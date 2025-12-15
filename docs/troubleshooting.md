
# Troubleshooting

This document captures common issues you might encounter when working with
Project 7 and how to resolve them.

---

## 1. Pending Pods in monitoring Namespace

**Symptom:**

- One or more pods in `monitoring` (e.g., Prometheus, Alertmanager) are `Pending`.
- `kubectl describe pod` shows messages like:
  `0/2 nodes are available: 2 Too many pods.`

**Cause:**

- The cluster does not have enough capacity to schedule additional pods.

**Resolution:**

- Use Terraform from Project 2 to increase the node group size or adjust min/max
  settings.
- Re-run `terraform apply`.
- After nodes join the cluster, the pods should move from `Pending` to `Running`.

---

## 2. ServiceMonitor Not Scraping Application

**Symptom:**

- Your app targets do not appear in Prometheus `/targets`.
- Dashboards do not show app data.

**Checks:**

1. Confirm the ServiceMonitor exists and references the expected namespaces:

   ```bash
   kubectl get servicemonitors -n monitoring
   kubectl describe servicemonitor project-app-servicemonitor -n monitoring
   ```

2. Confirm the application Services have the correct labels and port name:

   ```bash
   kubectl get svc -n project6-dev
   kubectl get svc -n project6-pre
   kubectl get svc -n project6-prod
   ```

   The Service should:
   - Use `name: http` for the port exposed on 80.
   - Include `app.kubernetes.io/name: project4-app` in `metadata.labels`.

3. Confirm `/metrics` is reachable from inside the cluster (e.g., using a debug pod).

**Fix:**

- Align labels and port names between Service and ServiceMonitor.
- Ensure `/metrics` is served by Nginx as configured.

---

## 3. Metrics 404 or Invalid Format

**Symptom:**

- Prometheus shows scrape errors for `/metrics`.
- Logs mention 404 or parse errors.

**Checks:**

1. Use `kubectl port-forward` or an ephemeral pod to curl the metrics endpoint:

   ```bash
   curl http://<service-ip>/metrics
   ```

2. Confirm the response:
   - Returns HTTP 200.
   - Matches Prometheus text exposition format.

**Fix:**

- Ensure `metrics.prom` is in the expected path inside the container.
- Verify the Nginx `location /metrics` block matches the container filesystem.

---

## 4. Dashboards Not Appearing in Grafana

**Symptom:**

- Custom dashboards do not show up in Grafana.

**Checks:**

1. Confirm ConfigMaps exist:

   ```bash
   kubectl get configmaps -n monitoring | grep dashboard
   ```

2. Confirm each ConfigMap:

   - Is in `monitoring` namespace.
   - Has `metadata.labels.grafana_dashboard: "1"`.
   - Has `data` keys ending with `.json`.

3. Check Argo CD Application status:
   - Ensure the Application managing monitoring manifests is **Synced**.

**Fix:**

- Add the missing label or correct the namespace.
- Fix JSON syntax if Grafana logs show parsing errors.
- Re-sync in Argo CD.

---

## 5. Argo CD OutOfSync for Monitoring

**Symptom:**

- The Argo CD Application for Project 7 shows `OutOfSync`.

**Checks:**

- Inspect the diff view to see which manifests differ.
- Common causes:
  - Manual `kubectl apply` modifying resources.
  - A resource created or changed outside of Git.

**Fix:**

- Prefer updating manifests in Git and letting Argo CD reconcile.
- If appropriate, use the **Sync** button with **Apply** and **Prune** enabled.

---

## 6. No Data in Dashboards

**Symptom:**

- Dashboards load but show `N/A` or `No data`.

**Checks:**

- Verify Prometheus is scraping the app metrics (`project_app_dummy_requests_total`).
- Use Prometheus or Grafana Explore to run the same queries used in panels.
- Confirm your time range includes the period when metrics are scraped.

**Fix:**

- Adjust the dashboard time range (e.g., last 1h).
- Confirm pods are running and `/metrics` is scraped successfully.
- Check for typos in label filters or metric names.

If these steps do not resolve the issue, work backward from Grafana → Prometheus
→ ServiceMonitor → Service → Pod logs.
