# References

This project was implemented using primary documentation for Prometheus, Grafana, Argo CD, and Kubernetes metrics.

---

## 1. Prometheus and Prometheus Operator

- Prometheus overview: https://prometheus.io/docs/introduction/overview/
- PromQL basics: https://prometheus.io/docs/prometheus/latest/querying/basics/
- PromQL functions: https://prometheus.io/docs/prometheus/latest/querying/functions/
- Prometheus Operator: https://github.com/prometheus-operator/prometheus-operator
- kube-prometheus-stack chart: https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

---

## 2. Grafana and Dashboards

- Grafana docs: https://grafana.com/docs/grafana/latest/
- Dashboards & panels: https://grafana.com/docs/grafana/latest/dashboards/
- Dashboard JSON model: https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/view-dashboard-json-model/
- Dashboard provisioning: https://grafana.com/docs/grafana/latest/administration/provisioning/#dashboards

---

## 3. Kubernetes Metrics and Exporters

- Resource metrics pipeline: https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/
- kube-state-metrics: https://github.com/kubernetes/kube-state-metrics
- node-exporter: https://github.com/prometheus/node_exporter

---

## 4. Argo CD and GitOps

- Argo CD docs: https://argo-cd.readthedocs.io/
- Application management: https://argo-cd.readthedocs.io/en/stable/user-guide/application_management/
- GitOps overview: https://www.weave.works/technologies/gitops/

---

## 5. Related Portfolio Projects

Project 7 extends earlier projects:

- **Project 1**: CI/CD to EKS; image extended here to expose `/metrics`
- **Project 4**: Helm chart for the application
- **Project 6**: ApplicationSet multi-environment deployment (dev/pre/prod)
