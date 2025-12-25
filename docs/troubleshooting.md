# Troubleshooting

This document captures common issues in Project 7 and a practical debug order:
Grafana → Prometheus → ServiceMonitor → Service → Pods → `/metrics`.

---

## 1. Pending Pods in `monitoring`

**Symptom**
- Prometheus/Grafana/Alertmanager pods stuck in `Pending`
- Scheduler events show capacity issues (e.g., “Too many pods”)

**Cause**
- Cluster capacity is insufficient (nodes, pod limits, or resources)

**Fix**
- Scale node group capacity (Project 2 Terraform), then re-check:

```bash
kubectl get pods -n monitoring
```

---

## 2. ServiceMonitor Not Scraping the App

**Symptom**
- App targets missing in Prometheus `/targets`
- Grafana app dashboards show no data

**Checks**

1. ServiceMonitor exists:

```bash
kubectl get servicemonitors -n monitoring
kubectl describe servicemonitor project-app-servicemonitor -n monitoring
```

2. App Services have expected labels and port name:

```bash
kubectl get svc -n project6-dev
kubectl get svc -n project6-pre
kubectl get svc -n project6-prod
```

Confirm:
- port name is `http`
- labels include `app.kubernetes.io/name: project4-app`

3. `/metrics` is reachable inside the cluster (debug pod).

**Fix**
- Align Service labels/port naming with ServiceMonitor selector/endpoints.

---

## 3. `/metrics` Returns 404 or Scrape Parse Errors

**Symptom**
- Prometheus shows scrape errors (404 or parse errors)

**Checks**
- Curl `/metrics` from inside the cluster and confirm:
  - HTTP 200
  - Prometheus text exposition format

**Fix**
- Verify `metrics.prom` path in the image
- Verify Nginx `/metrics` location and `try_files` path

---

## 4. Dashboards Not Appearing in Grafana

**Symptom**
- Custom dashboards do not show

**Checks**

1. ConfigMaps exist:

```bash
kubectl get configmaps -n monitoring | grep dashboard
```

2. Each dashboard ConfigMap:
- is in `monitoring`
- has label `grafana_dashboard: "1"`
- includes a `.json` key under `data`
- contains valid JSON

3. Argo CD Application is synced for monitoring assets.

---

## 5. Argo CD OutOfSync

**Symptom**
- Monitoring Application is `OutOfSync`

**Cause**
- Manual changes applied outside Git, or drift caused by defaults

**Fix**
- Prefer updating manifests in Git and syncing
- Use the Argo CD diff view to confirm intended changes

---

## 6. Dashboards Load but Show “No data”

**Symptom**
- Panels show N/A or “No data”

**Checks**
- Validate the underlying series in Prometheus or Grafana Explore
- Confirm the dashboard time range includes recent scrapes
- Confirm label filters (namespace/container names) match reality

If still stuck, debug backwards: Grafana → Prometheus → targets → ServiceMonitor → Service → Pods.
