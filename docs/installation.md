
# Installation

This document describes how to deploy the Project 7 observability stack and
confirm it is running correctly.

## 1. Prerequisites

You should have:

1. An EKS cluster created by **Project 2**.
2. Argo CD installed and configured (from Projects 3+).
3. The Project 6 ApplicationSet deploying the app into:
   - `project6-dev`
   - `project6-pre`
   - `project6-prod`
4. The Project 1 image updated (or rebuilt) to include the `/metrics` endpoint.

You should be using `kubectl` and `helm` against the EKS cluster context.

## 2. Ensure Monitoring CRDs Exist

Check that the Prometheus Operator CRDs are present (if not, they will be
installed by kube-prometheus-stack):

```bash
kubectl get crd | grep monitoring.coreos.com
```

You should see CRDs such as:

- `servicemonitors.monitoring.coreos.com`
- `prometheuses.monitoring.coreos.com`
- `prometheusrules.monitoring.coreos.com`
- `alertmanagers.monitoring.coreos.com`
- etc.

## 3. Create the monitoring Namespace

Apply the namespace manifest:

```bash
kubectl apply -f monitoring/namespace.yaml
```

Verify:

```bash
kubectl get ns monitoring
```

## 4. Deploy kube-prometheus-stack via Argo CD

Project 7 uses Argo CD to manage the Prometheus stack.

1. Apply the Argo CD Application manifest:

   ```bash
   kubectl apply -f argocd/apps/kube-prometheus-stack.yaml
   ```

2. In the Argo CD UI:
   - Locate the Application associated with Project 7.
   - Click **Sync** if it is not already synced.

3. Verify pods in the `monitoring` namespace:

   ```bash
   kubectl get pods -n monitoring
   ```

   You should see:
   - Prometheus
   - Grafana
   - Alertmanager
   - kube-state-metrics
   - node-exporter (DaemonSet)
   - Prometheus Operator

## 5. Deploy ServiceMonitor and Dashboards

Apply the ServiceMonitor and ensure dashboard ConfigMaps are present in Git so
Argo CD can sync them.

1. ServiceMonitor:

   ```bash
   kubectl apply -f monitoring/servicemonitors/project-app-servicemonitor.yaml
   ```

2. Dashboard ConfigMaps are in:

   ```text
   monitoring/dashboards/*.yaml
   ```

   Once committed and pushed, Argo CD will apply them.

## 6. Verification

### 6.1 Prometheus Targets

Port-forward Prometheus:

```bash
kubectl port-forward svc/kube-prometheus-stack-prometheus 9090:9090 -n monitoring
```

Open `http://localhost:9090/targets` and check:

- Application targets corresponding to:
  - project6-dev
  - project6-pre
  - project6-prod
- They should be in **UP** state.

### 6.2 Grafana Dashboards

Port-forward Grafana:

```bash
kubectl port-forward svc/kube-prometheus-stack-grafana 3000:80 -n monitoring
```

Open `http://localhost:3000` and:

- Navigate to **Dashboards → Browse**.
- Confirm that the following dashboards are present:
  - *App Health by Namespace*
  - *Cluster Overview*
  - *Application Metrics Detail*
  - *Workload Status (project4-app)*

If any of these steps fail, see `docs/troubleshooting.md`.
