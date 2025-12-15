
# Custom Metrics

This document explains how Project 7 adds a simple Prometheus metrics endpoint
to the Project 1 application and how that metric flows into Prometheus and Grafana.

## 1. Goals

- Demonstrate how to expose a Prometheus metrics endpoint from an application.
- Wire that endpoint into the existing Helm chart and EKS deployment.
- Consume the metric in Prometheus and visual dashboards.

The focus is on the wiring, not on complex business metrics.

## 2. Metrics Endpoint Design

Instead of implementing a full application with a custom metrics library, Project 7
uses a simple approach:

- A static metrics file `metrics.prom` is added to the container image.
- Nginx is configured to serve this file at `/metrics`.
- The format of the file follows Prometheus text exposition format.

Example `metrics.prom`:

```text
# HELP project_app_dummy_requests_total Dummy request counter for demo.
# TYPE project_app_dummy_requests_total counter
project_app_dummy_requests_total 1
```

This metric is used by dashboards in Project 7.

## 3. Nginx Configuration

The Nginx configuration is updated (for example, in `nginx/default.conf`) to
serve the metrics file:

```nginx
server {
    listen 80;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    location /metrics {
        root /usr/share/nginx/html;
        default_type text/plain;
        try_files /metrics/metrics.prom =404;
    }
}
```

This allows Prometheus to scrape `/metrics` on the application Service.

## 4. Kubernetes Service and Labels

The application Service (defined in the Project 4 chart) must satisfy:

- Exposes port 80 as `port: 80` with `name: http`.
- Uses consistent labels such as:

  ```yaml
  labels:
    app.kubernetes.io/name: project4-app
  ```

These labels are used by the ServiceMonitor to select the correct Services in
each namespace.

## 5. ServiceMonitor Configuration

The `ServiceMonitor` in `monitoring/servicemonitors/project-app-servicemonitor.yaml`
points Prometheus to the application Services:

- `namespaceSelector.matchNames` includes:
  - `project6-dev`
  - `project6-pre`
  - `project6-prod`

- `selector.matchLabels` targets the app Services:

  ```yaml
  selector:
    matchLabels:
      app.kubernetes.io/name: project4-app
  ```

- `endpoints` specify:

  ```yaml
  endpoints:
    - port: http
      path: /metrics
      interval: 30s
  ```

Once applied, Prometheus discovers and scrapes the `/metrics` endpoints.

## 6. Verifying the Metric

Use the Prometheus UI or Grafana Explore to verify:

```promql
project_app_dummy_requests_total
```

You should see time series per namespace (dev, pre, prod) once traffic or the
scrape has occurred. The dashboards use this metric for application-level views.
