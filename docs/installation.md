# Installation

## 1. Prerequisites

You should have:

1. An EKS cluster created by **Project 2**
2. Argo CD installed and working (Projects 3+)
3. Project 6 deploying the app into:
   - `project6-dev`
   - `project6-pre`
   - `project6-prod`
4. The application image updated to include `/metrics`

## 2. Confirm Monitoring CRDs

If kube-prometheus-stack has not been installed before, the chart will install required CRDs.
You can verify current CRDs:

```bash
kubectl get crd | grep monitoring.coreos.com
```

## 3. Create the `monitoring` Namespace

```bash
kubectl apply -f monitoring/namespace.yaml
kubectl get ns monitoring
```

## 4. Deploy kube-prometheus-stack via Argo CD

Apply the Argo CD Application:

```bash
kubectl apply -f argocd/apps/kube-prometheus-stack.yaml
```

In Argo CD, sync the Application if needed. Then verify:

```bash
kubectl get pods -n monitoring
```

You should see Prometheus, Grafana, Alertmanager, the operator, and exporters.

## 5. Deploy ServiceMonitor and Dashboards

- ServiceMonitor:

```bash
kubectl apply -f monitoring/servicemonitors/project-app-servicemonitor.yaml
```

- Dashboard ConfigMaps are stored under:

```text
monitoring/dashboards/*.yaml
```

Commit and push changes so Argo CD can reconcile.

## 6. Verification

### 6.1 Prometheus Targets

```bash
kubectl port-forward svc/kube-prometheus-stack-prometheus 9090:9090 -n monitoring
```

Open `http://localhost:9090/targets` and confirm the three `project6-*` targets are **UP**.

### 6.2 Grafana Dashboards

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
```

Open `http://localhost:3000` → **Dashboards → Browse** and confirm:

- App Health by Namespace
- Cluster Overview
- Application Metrics Detail
- Workload Status (project4-app)

If anything fails, see: `docs/troubleshooting.md`.
