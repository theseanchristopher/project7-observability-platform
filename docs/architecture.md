# Architecture

## 1. Overview

Project 7 adds a **Kubernetes observability platform** to the application stack built in Projects 1–6.
A dedicated `monitoring` namespace hosts the monitoring stack and observes:

- Kubernetes control plane and node metrics
- Kubernetes object state (Deployments, Pods, Nodes, etc.)
- Application metrics from Project 6 workloads in:
  - `project6-dev`
  - `project6-pre`
  - `project6-prod`

## 2. Components and Relationships

1. **Application workloads (Projects 1, 4, 6)**
   - Project 1 builds the container image (extended here to expose `/metrics`)
   - Project 4 defines the Helm chart (Deployment, Service, Ingress, HPA)
   - Project 6 deploys dev/pre/prod via Argo CD ApplicationSet

2. **Metrics endpoint**
   - The app serves `/metrics` via Nginx.
   - A simple static metric is exposed for demonstration:
     `project_app_dummy_requests_total`.

3. **Service + ServiceMonitor**
   - Each environment exposes a Service with `port: 80` named `http`.
   - A single `ServiceMonitor` in `monitoring` selects those Services across namespaces
     and scrapes `/metrics`.

4. **Prometheus Operator + Prometheus**
   - The operator reconciles CRDs (`Prometheus`, `ServiceMonitor`, `PrometheusRule`, etc.)
     into a running Prometheus instance.
   - Prometheus scrapes cluster/exporter metrics plus application targets discovered via ServiceMonitor.

5. **Grafana**
   - Uses Prometheus as a data source.
   - Loads custom dashboards provisioned from ConfigMaps in `monitoring/dashboards/`.

6. **GitOps (Argo CD)**
   - Argo CD reconciles the monitoring stack and assets from Git.
   - The monitoring platform remains reproducible and consistent across environments.

## 3. Data Flow Summary

1. App exposes `/metrics`.
2. Service exposes port `http`.
3. ServiceMonitor discovers Services across namespaces and scrapes `/metrics`.
4. Prometheus stores time series.
5. Grafana queries Prometheus using PromQL and renders dashboards.

## 4. Diagram

The diagram is stored at:

- `docs/images/project7-architecture.svg`

For stack details, see: `docs/monitoring-stack-overview.md`.
