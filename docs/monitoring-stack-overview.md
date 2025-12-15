
# Monitoring Stack Overview

This document describes the main components installed by the
`kube-prometheus-stack` Helm chart in the `monitoring` namespace and how they
relate to the rest of the platform.

## 1. Prometheus Operator

The **Prometheus Operator** simplifies running Prometheus and Alertmanager on
Kubernetes by managing them through custom resources (CRDs), including:

- `Prometheus`
- `Alertmanager`
- `ServiceMonitor`
- `PodMonitor`
- `PrometheusRule`

You do not manage Prometheus via direct `Deployment` manifests; instead, you
define a `Prometheus` resource and let the operator handle the details.

## 2. Prometheus

Prometheus is responsible for:

- Scraping metrics from:
  - Kubernetes control plane endpoints
  - kubelet / node-exporter
  - kube-state-metrics
  - Application endpoints discovered via ServiceMonitor
- Evaluating alerting rules
- Serving queries to Grafana and the Prometheus UI

Project 7 relies on Prometheus to scrape:

- `up{container="project4-app"}`
- `project_app_dummy_requests_total`
- Various Kubernetes and node metrics used in dashboards.

## 3. Grafana

Grafana provides the visualization layer:

- Uses Prometheus as a data source.
- Comes with pre-built dashboards for Kubernetes infrastructure.
- Loads custom dashboards from ConfigMaps labeled:

  ```yaml
  grafana_dashboard: "1"
  ```

Project 7 defines four custom dashboards:

1. App Health by Namespace  
2. Cluster Overview  
3. Application Metrics Detail  
4. Workload Status (project4-app)

These dashboards are managed as code in the repo and auto-loaded by Grafana.

## 4. Alertmanager

Alertmanager:

- Receives alerts fired by Prometheus.
- Groups, deduplicates, and routes alerts to receivers (e.g., email, Slack).

In Project 7:

- Alertmanager is deployed by kube-prometheus-stack.
- The platform is ready for alert routing configuration.
- For simplicity, no external receivers are configured, but you can add
  `PrometheusRule` resources and Alertmanager configuration later.

## 5. Exporters

The monitoring stack also includes:

- **kube-state-metrics**  
  - Exposes Kubernetes object metrics (Deployments, Pods, Nodes, etc.).

- **node-exporter**  
  - Exposes operating system-level metrics (CPU, memory, disk, etc.) for each node.

- **Kubelet metrics**  
  - Exposes container-level metrics used in CPU/memory dashboards.

These exporters feed Prometheus with the data used in the Cluster Overview and
Workload Status dashboards.

## 6. Integration with the Portfolio

Project 7 is not a standalone monitoring stack; it is specifically designed to
monitor the workloads created earlier:

- Project 1 builds the app.
- Project 4 defines the Helm chart.
- Project 6 deploys the app into dev / pre / prod.
- Project 7 observes and visualizes those deployments.

This ensures the monitoring story is tied directly to your existing portfolio.
