
# Alerts

Project 7 installs Alertmanager and supports Prometheus alerting rules, but keeps
alerting intentionally minimal so the focus remains on metrics and dashboards.

You can extend this project by adding `PrometheusRule` resources under
`monitoring/alerts/`.

## 1. PrometheusRule Basics

A `PrometheusRule` resource groups one or more alerting or recording rules.

Skeleton:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: project7-alerts
  namespace: monitoring
spec:
  groups:
    - name: project7.rules
      rules:
        - alert: ProjectAppDown
          expr: avg_over_time(up{container="project4-app"}[5m]) < 1
          for: 5m
          labels:
            severity: warning
          annotations:
            summary: "project4-app appears to be down"
            description: "No healthy pods for project4-app for the last 5 minutes."
```

Once applied, Prometheus evaluates these rules and sends alerts to Alertmanager.

## 2. Example Alerts

You might consider:

1. **Application Down**

   As shown above, checks if all `project4-app` pods are down.

2. **High Pod Restart Rate**

   ```promql
   sum by (namespace) (
     rate(kube_pod_container_status_restarts_total{container="project4-app"}[10m])
   ) > 0
   ```

3. **High CPU or Memory Usage**

   Use container CPU and memory metrics to trigger alerts when usage exceeds a
   threshold over a sustained period.

## 3. Alertmanager Configuration

By default, kube-prometheus-stack deploys an Alertmanager instance with a basic
configuration. To send alerts to external systems (Slack, email, etc.), you
would update the Alertmanager configuration (often via a `Secret` or the chart
values) and add appropriate receivers and routes.

Project 7 leaves this step out to keep the scope manageable, but the platform is
ready to support it.
