# Project 7 — Kubernetes Observability Platform (Prometheus, Grafana, Alerting)

Project 7 adds a **production-style observability layer** to the Kubernetes platform built in Projects 1–6.
It deploys the **kube-prometheus-stack** (Prometheus Operator, Prometheus, Alertmanager, Grafana, and exporters)
into a dedicated `monitoring` namespace and configures it to scrape **application metrics** from the
multi-environment workloads deployed in **Project 6** (`project6-dev`, `project6-pre`, `project6-prod`).

This project is intentionally focused on the **platform wiring** (discovery, scraping, dashboards-as-code, GitOps flow),
not advanced application instrumentation.

---

## 1. What This Project Demonstrates

- Cluster-level monitoring using Prometheus Operator and standard exporters
- Application-level metrics collected from a `/metrics` endpoint
- A `ServiceMonitor` scraping the app across **three namespaces**
- Grafana dashboards managed as code and provisioned via ConfigMaps
- A GitOps workflow where Argo CD reconciles the monitoring stack and assets

---

## 2. High-Level Architecture

This project builds on earlier portfolio components:

- **Project 1**: builds the application container image (extended here to expose `/metrics`)
- **Project 4**: provides the Helm chart (Deployment, Service, Ingress, HPA) for the application
- **Project 6**: deploys the application into `project6-dev`, `project6-pre`, and `project6-prod` via ApplicationSet
- **Project 7 (this repo)**: installs the monitoring stack and scrapes/visualizes those workloads

Architecture diagram:

- `docs/images/project7-architecture.svg`

For the detailed flow, see: `docs/architecture.md`.

---

## 3. Repository Structure

```text
project7-observability-platform/
├── README.md
├── argocd/
│   └── apps/
│       └── kube-prometheus-stack.yaml
├── monitoring/
│   ├── namespace.yaml
│   ├── servicemonitors/
│   │   └── project-app-servicemonitor.yaml
│   ├── dashboards/
│   │   ├── app-health-dashboard.yaml
│   │   ├── cluster-overview-dashboard.yaml
│   │   ├── app-metrics-dashboard.yaml
│   │   └── workload-status-dashboard.yaml
│   └── alerts/
│       └── (optional PrometheusRule manifests)
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

---

## 4. Monitoring Stack Components

Project 7 installs `kube-prometheus-stack`, which includes:

1. **Prometheus Operator** (CRDs + controllers for Prometheus/Alertmanager)
2. **Prometheus** (scrapes targets and stores time series)
3. **Grafana** (visualization, dashboard provisioning via sidecar)
4. **Alertmanager** (receives and routes alerts)
5. **Exporters** (`kube-state-metrics`, `node-exporter`, kubelet metrics)

See: `docs/monitoring-stack-overview.md`.

---

## 5. Application Metrics Integration

To demonstrate end-to-end observability, the application image exposes a basic Prometheus endpoint:

- `/metrics` serves a static `metrics.prom` file (Prometheus text format)
- Example metric:

```text
project_app_dummy_requests_total 1
```

This is intentionally simple; the goal is to show the complete pipeline:

`App → /metrics → ServiceMonitor → Prometheus → Grafana`

See: `docs/custom-metrics.md`.

---

## 6. ServiceMonitor and Scrape Targets

A single `ServiceMonitor` in `monitoring` is configured to:

- select Services labeled `app.kubernetes.io/name: project4-app`
- scrape `/metrics` on the `http` port
- discover targets across:
  - `project6-dev`
  - `project6-pre`
  - `project6-prod`

See: `docs/custom-metrics.md` and `docs/installation.md`.

---

## 7. Dashboards as Code (GitOps)

Custom dashboards are stored as ConfigMaps under `monitoring/dashboards/` and labeled:

```yaml
labels:
  grafana_dashboard: "1"
```

Grafana’s dashboard sidecar automatically loads these dashboards from ConfigMaps in the `monitoring` namespace.

Dashboards included:

1. **App Health by Namespace**
2. **Cluster Overview**
3. **Application Metrics Detail**
4. **Workload Status (project4-app)**

See: `docs/dashboards.md`.

---

## 8. Deployment Flow (GitOps)

1. Commit monitoring manifests to this repo
2. Argo CD reconciles:
   - the Helm-based monitoring stack
   - ServiceMonitor(s)
   - dashboard ConfigMaps
   - optional PrometheusRules
3. Prometheus discovers targets via ServiceMonitor and begins scraping
4. Grafana renders dashboards using PromQL queries

---

## 9. Verification (Quick Checklist)

### 9.1 Pods Running

```bash
kubectl get pods -n monitoring
```

Expect to see Prometheus, Grafana, Alertmanager, exporters, and the operator.

### 9.2 Prometheus Targets

```bash
kubectl port-forward svc/kube-prometheus-stack-prometheus 9090:9090 -n monitoring
```

Open `http://localhost:9090/targets` and confirm the app targets for `project6-*` are **UP**.

### 9.3 Grafana Dashboards

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
```

Open `http://localhost:3000` → **Dashboards → Browse** and confirm the four project dashboards exist.

---

## 10. Troubleshooting and References

- Troubleshooting: `docs/troubleshooting.md`
- References: `docs/references.md`
