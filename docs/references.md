
# References

This document lists key documentation and reference material used when
implementing Project 7.

---

## 1. Prometheus and Prometheus Operator

- **Prometheus overview and documentation**  
  https://prometheus.io/docs/introduction/overview/

- **PromQL querying basics**  
  https://prometheus.io/docs/prometheus/latest/querying/basics/

- **PromQL functions and operators**  
  https://prometheus.io/docs/prometheus/latest/querying/functions/

- **Prometheus Operator** (manages Prometheus, Alertmanager, ServiceMonitor, etc.)  
  https://github.com/prometheus-operator/prometheus-operator

- **kube-prometheus-stack Helm chart**  
  https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack

---

## 2. Grafana and Dashboards

- **Grafana documentation**  
  https://grafana.com/docs/grafana/latest/

- **Dashboards and panels**  
  https://grafana.com/docs/grafana/latest/dashboards/

- **Dashboard JSON model** (used for ConfigMap embedding)  
  https://grafana.com/docs/grafana/latest/dashboards/build-dashboards/view-dashboard-json-model/

- **Provisioning dashboards via files**  
  https://grafana.com/docs/grafana/latest/administration/provisioning/#dashboards

---

## 3. Kubernetes Metrics and Exporters

- **Kubernetes metrics pipeline overview**  
  https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/

- **kube-state-metrics**  
  https://github.com/kubernetes/kube-state-metrics

- **node-exporter**  
  https://github.com/prometheus/node_exporter

---

## 4. Argo CD and GitOps

- **Argo CD documentation**  
  https://argo-cd.readthedocs.io/

- **Managing Applications with Argo CD**  
  https://argo-cd.readthedocs.io/en/stable/user-guide/application_management/

- **GitOps concept overview**  
  https://www.weave.works/technologies/gitops/

---

## 5. Related Portfolio Projects

Project 7 depends on and extends several earlier projects:

- **Project 1 – CI/CD to EKS with GitHub Actions and Nginx app**  
  Builds the application image and (in this project) exposes `/metrics`.  

- **Project 4 – Helm chart for the application**  
  Defines Deployment, Service, Ingress, and HPA for the app.  

- **Project 6 – ApplicationSet multi-environment deployment**  
  Deploys the application into `project6-dev`, `project6-pre`, and
  `project6-prod`, which are monitored by Project 7.  

