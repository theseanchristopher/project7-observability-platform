
# Architecture

## 1. High-Level Overview

Project 7 adds a **Kubernetes observability platform** to the application stack
built in Projects 1–6. It introduces a dedicated `monitoring` namespace that
hosts:

- Prometheus Operator (via kube-prometheus-stack)
- Prometheus
- Alertmanager
- Grafana
- kube-state-metrics
- node-exporter

The stack monitors:

- Core Kubernetes components
- Node and container-level metrics
- Application metrics from the Project 6 workloads running in:
  - `project6-dev`
  - `project6-pre`
  - `project6-prod`

## 2. Components and Relationships

At a high level:

1. **Application Workloads (Projects 1, 4, 6)**  
   - Project 1 builds the container image.
   - Project 4 defines the Helm chart (Deployment, Service, Ingress, HPA).
   - Project 6 deploys the app into dev / pre / prod environments using an
     Argo CD ApplicationSet.

2. **Metrics Endpoint**  
   - The Project 1 app image is extended to serve a `/metrics` endpoint via Nginx.
   - A static file `metrics.prom` exposes the metric
     `project_app_dummy_requests_total`.

3. **Service and ServiceMonitor**  
   - Each environment has a Service that selects the app pods and exposes port 80
     as `http`.
   - A single `ServiceMonitor` in the `monitoring` namespace selects these Services
     across all three namespaces and scrapes `/metrics`.

4. **Prometheus Operator and Prometheus**  
   - The Prometheus Operator reconciles the `Prometheus` and `ServiceMonitor`
     resources into a running Prometheus instance.
   - Prometheus scrapes:
     - Application metrics via ServiceMonitor
     - Kubernetes API server, controller manager, scheduler, etc.
     - kube-state-metrics
     - node-exporter

5. **Grafana**  
   - Uses Prometheus as a data source.
   - Loads both:
     - Built-in Kubernetes dashboards.
     - Custom dashboards provisioned via ConfigMaps in `monitoring/dashboards/`.

6. **GitOps and Argo CD**  
   - An Argo CD Application points at this repo.
   - kube-prometheus-stack, ServiceMonitor, and dashboards are all reconciled
     from Git.

## 3. Diagram

The architecture diagram is stored at:

- `docs/images/project7-architecture.svg`

Conceptually, it shows:

- Project 6 namespaces (`project6-dev`, `project6-pre`, `project6-prod`) with
  app pods and Services.
- A `ServiceMonitor` in `monitoring` selecting those Services.
- Prometheus scraping metrics via the ServiceMonitor.
- Grafana querying Prometheus and rendering dashboards.
- Argo CD reconciling all monitoring-related manifests from the Project 7 repo.

## 4. Data Flow Summary

1. Application exposes `/metrics` via Nginx.
2. Kubernetes Service exposes the app on `port: 80` named `http`.
3. ServiceMonitor selects the Services and scrapes `/metrics`.
4. Prometheus scrapes the targets and stores the time series.
5. Grafana queries Prometheus using PromQL.
6. Dashboards render health, resource usage, and custom app metrics.

For more details on the monitoring tools themselves, see
`docs/monitoring-stack-overview.md`.
