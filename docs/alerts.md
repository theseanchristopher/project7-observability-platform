# Alerts

Project 7 installs Alertmanager and supports Prometheus alerting rules, but keeps alerting intentionally minimal.
The primary goal is to demonstrate the platform wiring; alert routing (Slack/email) is a natural extension.

## 1. PrometheusRule Basics

A `PrometheusRule` groups alerting (and optionally recording) rules evaluated by Prometheus.

Example skeleton:

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

## 2. Example Alerts

1. **Application Down**  
   Detects loss of availability over a sustained window.

2. **High Pod Restart Rate**

```promql
sum by (namespace) (
  rate(kube_pod_container_status_restarts_total{container="project4-app"}[10m])
) > 0
```

3. **High CPU or Memory Usage**  
   Use container CPU/memory metrics with a sustained threshold window.

## 3. Alertmanager Configuration

kube-prometheus-stack deploys Alertmanager with a basic configuration.
To route alerts to external receivers, you would:

- Configure receivers and routes (commonly via chart values and a Secret)
- Add PrometheusRules under `monitoring/alerts/`

Project 7 leaves external integrations out-of-scope to keep the project focused.
