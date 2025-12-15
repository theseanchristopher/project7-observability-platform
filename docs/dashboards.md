
# Dashboards

This document explains how Grafana dashboards are managed as code in Project 7
and describes the four dashboards included in the observability platform.

---

## 1. Overview

Project 7 uses **ConfigMap-based dashboard provisioning**:

- The kube-prometheus-stack Helm chart enables a Grafana sidecar.
- The sidecar watches for ConfigMaps in the `monitoring` namespace with:

  ```yaml
  labels:
    grafana_dashboard: "1"
  ```

- Each ConfigMap embeds a Grafana dashboard JSON model under `data`.
- Argo CD applies these ConfigMaps from this Git repository.

This makes dashboards:

- Version-controlled
- Reviewed via pull requests
- Synchronized across clusters via GitOps

---

## 2. Creating Dashboards in Grafana

The recommended workflow is:

1. **Design dashboard in Grafana UI.**
2. **Export the JSON model.**
3. **Wrap it in a ConfigMap and commit to Git.**

### 2.1 Access Grafana

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
```

Open `http://localhost:3000`.

### 2.2 Create or Edit a Dashboard

1. In the left sidebar, select:

   **Dashboards → New → New dashboard**

2. Click **Add a new panel**.
3. Select the **Prometheus** data source.
4. Enter a PromQL query (for example):

   ```promql
   avg by (namespace) (up{container="project4-app"})
   ```

5. Choose a visualization (e.g., Bar gauge).
6. Configure thresholds, units, legends, etc.
7. Click **Apply**.
8. Repeat for additional panels as needed.
9. Click **Save dashboard**, give it a meaningful name, and save.

At this point, the dashboard exists in Grafana but is not yet managed by Git.

---

## 3. Exporting the Dashboard JSON Model

To manage the dashboard via GitOps:

1. Open the dashboard in Grafana.
2. Click the **gear icon** (Dashboard settings).
3. In the left menu, click **JSON model**.
4. Click **Copy to clipboard**.
5. Paste the JSON into a local file, e.g.:

   ```bash
   app-health-dashboard.json
   ```

This JSON is the canonical definition of the dashboard and is what we embed in
a ConfigMap.

---

## 4. Wrapping JSON in a ConfigMap

Create a ConfigMap manifest in `monitoring/dashboards/`.

Example skeleton:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-health-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "1"
data:
  app-health-dashboard.json: |
    { ... JSON copied from Grafana ... }
```

Key points:

- `namespace: monitoring` matches where Grafana runs.
- `grafana_dashboard: "1"` tells the sidecar to load the dashboard.
- The key under `data` ends in `.json`.
- The value is the exact JSON model copied from Grafana.

After committing and pushing this file, Argo CD applies the ConfigMap, and the
Grafana sidecar automatically loads the dashboard.

---

## 5. Dashboards Implemented in Project 7

Project 7 includes **four** dashboards, each defined in `monitoring/dashboards/`.

### 5.1 App Health by Namespace

- **ConfigMap name:** `app-health-dashboard`
- **Grafana title:** `App Health by Namespace`
- **Purpose:** High-level visibility into application health across the three
  Project 6 namespaces.

Key query:

```promql
avg by (namespace) (up{container="project4-app"})
```

Interpretation:

- `1` means all pods in that namespace are reporting as UP.
- `0` means none are UP.
- Intermediate values (e.g., `0.5`) indicate partial availability.

Visualization: bar gauge panel grouped by namespace.

---

### 5.2 Cluster Overview

- **ConfigMap name:** `cluster-overview-dashboard`
- **Grafana title:** `Cluster Overview`
- **Purpose:** Summarizes cluster capacity and utilization.

Example panels / queries:

- **Node count:**

  ```promql
  count(kube_node_info)
  ```

- **Running pods:**

  ```promql
  sum(kube_pod_status_phase{phase="Running"})
  ```

- **Total CPU usage (cores):**

  ```promql
  sum(rate(container_cpu_usage_seconds_total{container!=""}[5m]))
  ```

- **Cluster memory usage (bytes):**

  ```promql
  sum(container_memory_working_set_bytes{container!=""})
  ```

These panels allow you to answer: Is the cluster healthy and sized appropriately?

---

### 5.3 Application Metrics Detail

- **ConfigMap name:** `app-metrics-dashboard`
- **Grafana title:** `Application Metrics Detail`
- **Purpose:** Focus on the custom metric exposed at `/metrics`.

Key metric:

```promql
project_app_dummy_requests_total
```

Example views:

- Total dummy requests across all namespaces:

  ```promql
  sum(project_app_dummy_requests_total)
  ```

- Requests per namespace over time:

  ```promql
  sum by (namespace) (project_app_dummy_requests_total)
  ```

- Table of raw time series:

  ```promql
  project_app_dummy_requests_total
  ```

This dashboard demonstrates the wiring for app-specific metrics.

---

### 5.4 Workload Status (project4-app)

- **ConfigMap name:** `workload-status-dashboard`
- **Grafana title:** `Workload Status (project4-app)`
- **Purpose:** Visualize deployment health for the `project4-app` Deployment
  across environments.

Example queries:

- **Desired replicas:**

  ```promql
  sum by (namespace) (
    kube_deployment_status_replicas{deployment="project4-app"}
  )
  ```

- **Available replicas:**

  ```promql
  sum by (namespace) (
    kube_deployment_status_replicas_available{deployment="project4-app"}
  )
  ```

- **Restart rate:**

  ```promql
  sum by (namespace) (
    rate(kube_pod_container_status_restarts_total{container="project4-app"}[5m])
  )
  ```

This dashboard helps answer: Is my app fully rolled out and stable?

---

## 6. Keeping Dashboards in Sync

When you modify dashboards:

1. Edit the dashboard in Grafana.
2. Export the JSON model again.
3. Update the corresponding ConfigMap in `monitoring/dashboards/*.yaml`.
4. Commit and push.
5. Let Argo CD sync the changes.

This ensures Grafana dashboards are always reproducible and version-controlled.

