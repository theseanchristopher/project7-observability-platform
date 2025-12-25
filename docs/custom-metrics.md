# Custom Metrics

## 1. Goals

- Expose a Prometheus metrics endpoint from the application
- Wire the endpoint into the existing Helm chart and multi-environment deployment
- Query the metric in Prometheus and Grafana dashboards

The focus is the **end-to-end plumbing**, not complex business metrics.

## 2. Metrics Endpoint Design

Project 7 uses a simple approach:

- Add a static metrics file (`metrics.prom`) into the container image
- Configure Nginx to serve the file at `/metrics`
- Use the Prometheus text exposition format

Example `metrics.prom`:

```text
# HELP project_app_dummy_requests_total Dummy request counter for demo.
# TYPE project_app_dummy_requests_total counter
project_app_dummy_requests_total 1
```

## 3. Nginx Configuration

Example Nginx snippet:

```nginx
location /metrics {
    root /usr/share/nginx/html;
    default_type text/plain;
    try_files /metrics/metrics.prom =404;
}
```

## 4. Kubernetes Service Requirements

The Service (from the Project 4 chart) must:

- Expose port 80 as `name: http`
- Include labels used by the ServiceMonitor, for example:

```yaml
labels:
  app.kubernetes.io/name: project4-app
```

## 5. ServiceMonitor Configuration

The ServiceMonitor (`monitoring/servicemonitors/project-app-servicemonitor.yaml`) configures:

- `namespaceSelector.matchNames`: `project6-dev`, `project6-pre`, `project6-prod`
- `selector.matchLabels`: selects the app Service
- `endpoints`: scrapes `port: http`, `path: /metrics`, `interval: 30s`

## 6. Verifying the Metric

Use Prometheus or Grafana Explore:

```promql
project_app_dummy_requests_total
```

If you do not see data, work backward:

Grafana → Prometheus → ServiceMonitor → Service → Pod → `/metrics`

See also: `docs/troubleshooting.md`.
