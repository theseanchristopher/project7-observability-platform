
# Project 7 — Kubernetes Observability Platform (Prometheus, Grafana, Alerting)

This project adds a **production-style observability layer** on top of the Kubernetes
platform created in Projects 1–6. It installs the `kube-prometheus-stack` Helm chart
(Prometheus Operator, Prometheus, Alertmanager, Grafana, exporters) into a dedicated
`monitoring` namespace and wires it to scrape **application metrics** from the multi-
environment deployment created in **Project 6** (`project6-dev`, `project6-pre`,
`project6-prod`).

Project 7 demonstrates:

- Cluster-level monitoring with **Prometheus Operator** and exporters
- Application-level metrics via a `/metrics` endpoint added to the Project 1 app
- A **ServiceMonitor** that discovers and scrapes the app in dev / pre / prod
- **GitOps-managed Grafana dashboards** provisioned from ConfigMaps
- Documentation for installation, custom metrics, dashboards, alerts, and troubleshooting

For deeper technical details, see the `/docs` directory.

---

## 1. High-Level Architecture

The observability stack builds on the existing portfolio:

- **Project 1**  
  Builds and pushes the application container image to ECR. In Project 7, the image
  is extended with a `/metrics` endpoint.

- **Project 4**  
  Provides the Helm chart (Deployment, Service, Ingress, HPA) for the application.

- **Project 6**  
  Deploys the application into three namespaces using Argo CD ApplicationSet:  
  `project6-dev`, `project6-pre`, `project6-prod`.

- **Project 7 (this project)**  
  Installs `kube-prometheus-stack` into the `monitoring` namespace and configures
  Prometheus to scrape application metrics in all three namespaces. Grafana dashboards
  are managed as code and loaded automatically.

The architecture diagram is stored at:

- `docs/images/project7-architecture.svg`

---

## 2. Repository Structure

A typical layout for this project looks like:

```text
project7-observability-platform/
├── README.md
├── argocd/
│   └── apps/
│       └── kube-prometheus-stack.yaml        # Argo CD Application for the Helm chart
├── monitoring/
│   ├── namespace.yaml                        # monitoring namespace
│   ├── servicemonitors/
│   │   └── project-app-servicemonitor.yaml   # scrapes app metrics in dev/pre/prod
│   ├── dashboards/
│   │   ├── app-health-dashboard.yaml
│   │   ├── cluster-overview-dashboard.yaml
│   │   ├── app-metrics-dashboard.yaml
│   │   └── workload-status-dashboard.yaml
│   └── alerts/
│       └── (optional PrometheusRule files)
└── docs/
    ├── architecture.md
    ├── monitoring-stack-overview.md
    ├── installation.md
    ├── custom-metrics.md
    ├── dashboards.md
    ├── alerts.md
    ├── troubleshooting.md
    ├── references.md
    └── images/
        └── project7-architecture.svg
```

This structure mirrors the previous projects: a top-level README, a `/docs` folder
for deep dives, and GitOps manifests organized by concern.

---

## 3. Monitoring Stack Components

Project 7 uses the `kube-prometheus-stack` chart to install and manage:

1. **Prometheus Operator**  
   - Manages Prometheus and Alertmanager instances
   - Provides CRDs: `Prometheus`, `Alertmanager`, `ServiceMonitor`, `PodMonitor`,
     `PrometheusRule`, and more

2. **Prometheus**  
   - Scrapes cluster metrics (API server, controller manager, scheduler, etc.)
   - Scrapes node metrics via node-exporter
   - Scrapes object metrics via kube-state-metrics
   - Scrapes application metrics via `project-app-servicemonitor`

3. **Grafana**  
   - Comes with built-in Kubernetes dashboards
   - Loads custom dashboards from ConfigMaps labeled `grafana_dashboard: "1"`
   - Shows application health and workload status via Project 7 dashboards

4. **Alertmanager**  
   - Deployed by the chart
   - Ready to receive alerts from Prometheus
   - Not wired to external receivers (Slack, email, etc.) in this project, but can
     be extended later

5. **Exporters**  
   - `kube-state-metrics` for Kubernetes object state
   - `node-exporter` for node-level metrics
   - Kubelet metrics for container CPU/memory usage

A more detailed explanation is provided in `docs/monitoring-stack-overview.md`.

---

## 4. Application Metrics Integration

To demonstrate application-level observability, Project 7 adds a very simple
Prometheus endpoint to the Project 1 application:

- A static file `metrics.prom` is added under the app source tree.
- Nginx is configured to serve this file at the `/metrics` path.
- The application’s Kubernetes Service exposes port `http` (port 80), which is
  referenced by Prometheus.

Example metric exposed:

```text
project_app_dummy_requests_total 1
```

This metric is intentionally simple; the goal is to show the full path from:

`Application → /metrics → ServiceMonitor → Prometheus → Grafana dashboards`

More details are available in `docs/custom-metrics.md`.

---

## 5. ServiceMonitor and Scrape Targets

Project 7 defines a `ServiceMonitor` that instructs Prometheus to discover and
scrape the application in all three environments:

- `project6-dev`
- `project6-pre`
- `project6-prod`

Key aspects of `project-app-servicemonitor.yaml`:

- `namespaceSelector.matchNames` selects the three namespaces
- `selector.matchLabels` targets Services with `app.kubernetes.io/name: project4-app`
- `endpoints` defines:
  - `port: http`
  - `path: /metrics`
  - `interval: 30s`

Once applied, Prometheus shows the app targets as **UP** in the `/targets` UI, and
Grafana can query them using PromQL.

---

## 6. GitOps-Managed Grafana Dashboards

All Grafana dashboards in Project 7 are managed as code. Each dashboard is
defined as a `ConfigMap` in `monitoring/dashboards/` with:

```yaml
metadata:
  namespace: monitoring
  labels:
    grafana_dashboard: "1"
data:
  some-dashboard.json: |
    { ... full Grafana dashboard JSON ... }
```

The `kube-prometheus-stack` chart enables Grafana’s dashboard sidecar, which:

- Watches for ConfigMaps in `monitoring`
- Detects the `grafana_dashboard: "1"` label
- Loads the embedded JSON as a Grafana dashboard

Dashboards implemented in this project:

1. **App Health by Namespace**  
   - Shows `avg by (namespace) (up{container="project4-app"})` as a bar gauge

2. **Cluster Overview**  
   - Node count, running pods, cluster CPU and memory time series

3. **Application Metrics Detail**  
   - `project_app_dummy_requests_total` aggregated by namespace and over time

4. **Workload Status (project4-app)**  
   - Desired vs. available replicas
   - Pod restart rates per namespace

See `docs/dashboards.md` for a full explanation of how to build, export, and
manage these dashboards.

---

## 7. Deployment and GitOps Flow

Project 7 follows the same GitOps principles used in earlier projects:

1. **Edit configuration** in the Project 7 repo:
   - `argocd/apps/kube-prometheus-stack.yaml`
   - `monitoring/namespace.yaml`
   - `monitoring/servicemonitors/project-app-servicemonitor.yaml`
   - `monitoring/dashboards/*.yaml`
   - `monitoring/alerts/*.yaml` (optional)

2. **Commit and push** to the Git repository.

3. **Argo CD**:
   - Watches the Project 7 repo for changes
   - Reconciles the desired state into the cluster
   - Ensures kube-prometheus-stack, ServiceMonitor, dashboards, and alerts
     are all kept in sync

4. **Prometheus and Grafana**:
   - Prometheus automatically discovers new scrape targets via ServiceMonitor
   - Grafana automatically loads or updates dashboards from ConfigMaps

No manual `kubectl apply` steps to production are required.

---

## 8. Verification Steps

After applying the stack and confirming that node capacity is sufficient:

1. **Check monitoring namespace:**

   ```bash
   kubectl get pods -n monitoring
   ```

   You should see:
   - Prometheus
   - Alertmanager
   - Grafana
   - kube-state-metrics
   - node-exporter
   - Prometheus Operator

2. **Port-forward Grafana:**

   ```bash
   kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
   ```

   Open `http://localhost:3000`.

3. **Verify dashboards:**

   - Go to **Dashboards → Browse**
   - Confirm dashboards:
     - *App Health by Namespace*
     - *Cluster Overview*
     - *Application Metrics Detail*
     - *Workload Status (project4-app)*

4. **Verify app metrics in Prometheus:**

   Port-forward Prometheus:

   ```bash
   kubectl port-forward svc/kube-prometheus-stack-prometheus 9090:9090 -n monitoring
   ```

   Visit `http://localhost:9090` and run:

   ```promql
   up{container="project4-app"}
   project_app_dummy_requests_total
   ```

   You should see time series for the three namespaces.

---

## 9. Troubleshooting

Common issues and quick checks:

- **Pending Prometheus or Grafana pods**  
  - Cause: Not enough node capacity / Too many pods on the cluster  
  - Fix: Adjust node group size in Terraform (Project 2) and re-apply

- **ServiceMonitor not scraping**  
  - Check that the Service:
    - Uses `port: http` (and that it matches the ServiceMonitor endpoint)
    - Has labels matching the ServiceMonitor `selector.matchLabels`
  - Check that `/metrics` returns a 200 response inside the cluster

- **Dashboards not appearing**  
  - Ensure the ConfigMap:
    - Is in `monitoring` namespace
    - Has `grafana_dashboard: "1"` label
    - Contains valid JSON under a key ending with `.json`

- **Argo CD OutOfSync**  
  - Check the Argo CD Application for this project
  - Inspect the diff and reconcile manually if necessary

Additional details and extended troubleshooting are in `docs/troubleshooting.md`.

---

## 10. References

See `docs/references.md` for links to:

- Prometheus and PromQL documentation
- Prometheus Operator and kube-prometheus-stack
- Grafana dashboards and JSON model
- Argo CD and GitOps concepts
- Kubernetes metrics pipeline and exporters
- Related repositories for Projects 1, 4, and 6
