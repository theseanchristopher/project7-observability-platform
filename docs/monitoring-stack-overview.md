# Monitoring Stack Overview

## 1. Prometheus Operator

The Prometheus Operator runs Prometheus and Alertmanager using CRDs, including:

- `Prometheus`
- `Alertmanager`
- `ServiceMonitor`
- `PodMonitor`
- `PrometheusRule`

This removes the need to manage Prometheus and Alertmanager as raw Deployments.

## 2. Prometheus

Prometheus:

- Scrapes metrics from:
  - Kubernetes control plane endpoints
  - kubelet / node-exporter
  - kube-state-metrics
  - application endpoints discovered via ServiceMonitor
- Evaluates alerting rules (PrometheusRules)
- Serves queries to Grafana and the Prometheus UI

In this project, Prometheus scrapes application series such as:

- `up{container="project4-app"}`
- `project_app_dummy_requests_total`

## 3. Grafana

Grafana provides the visualization layer:

- Prometheus is the primary data source
- Built-in Kubernetes dashboards are included
- Custom dashboards are loaded from ConfigMaps labeled:

```yaml
grafana_dashboard: "1"
```

Project 7 includes four custom dashboards (see `docs/dashboards.md`).

## 4. Alertmanager

Alertmanager receives alerts fired by Prometheus and can route them to receivers
(email, Slack, PagerDuty, etc.).

In Project 7:

- Alertmanager is deployed by kube-prometheus-stack
- External receivers are intentionally out of scope
- The platform is ready for future routing configuration

## 5. Exporters

Key exporters include:

- `kube-state-metrics` (Kubernetes object metrics)
- `node-exporter` (node OS metrics)
- kubelet metrics (container CPU/memory metrics)

## 6. Integration with the Portfolio

Project 7 observes workloads built earlier:

- Project 1 builds the app image
- Project 4 defines the Helm chart
- Project 6 deploys multi-environment workloads
- Project 7 monitors and visualizes those workloads
