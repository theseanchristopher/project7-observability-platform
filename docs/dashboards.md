# Dashboards

## 1. Overview

Project 7 manages Grafana dashboards as code using ConfigMaps:

- kube-prometheus-stack enables a Grafana dashboard sidecar
- The sidecar watches for ConfigMaps in `monitoring` labeled:

```yaml
labels:
  grafana_dashboard: "1"
```

- Each ConfigMap embeds a Grafana dashboard JSON model under `data`
- Argo CD applies these ConfigMaps from Git

This makes dashboards reproducible, reviewable, and portable.

## 2. Workflow: Create → Export → Commit

1. Create or update a dashboard in the Grafana UI
2. Export the **JSON model**
3. Wrap the JSON in a ConfigMap under `monitoring/dashboards/`
4. Commit and push so Argo CD can sync

### 2.1 Access Grafana

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
```

### 2.2 Export JSON Model

Dashboard → Settings (gear icon) → JSON model → Copy.

## 3. ConfigMap Pattern

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
    { ... JSON ... }
```

Key points:

- ConfigMap must be in `monitoring`
- Must include `grafana_dashboard: "1"`
- `data` key should end in `.json`
- JSON must be valid

## 4. Dashboards Included

### 4.1 App Health by Namespace

Query:

```promql
avg by (namespace) (up{container="project4-app"})
```

Interpretation: `1` = all targets up, `0` = none up.

### 4.2 Cluster Overview

Example panels/queries:

- Node count: `count(kube_node_info)`
- Running pods: `sum(kube_pod_status_phase{phase="Running"})`
- CPU cores: `sum(rate(container_cpu_usage_seconds_total{container!=""}[5m]))`
- Memory bytes: `sum(container_memory_working_set_bytes{container!=""})`

### 4.3 Application Metrics Detail

Key metric:

```promql
project_app_dummy_requests_total
```

Example aggregations:

- Total: `sum(project_app_dummy_requests_total)`
- By namespace: `sum by (namespace) (project_app_dummy_requests_total)`

### 4.4 Workload Status (project4-app)

Example queries:

- Desired replicas:
  `sum by (namespace) (kube_deployment_status_replicas{deployment="project4-app"})`
- Available replicas:
  `sum by (namespace) (kube_deployment_status_replicas_available{deployment="project4-app"})`
- Restart rate:
  `sum by (namespace) (rate(kube_pod_container_status_restarts_total{container="project4-app"}[5m]))`

## 5. Keeping Dashboards in Sync

When dashboards change:

1. Update in Grafana
2. Re-export JSON
3. Replace JSON in the ConfigMap
4. Commit and push
5. Sync via Argo CD
