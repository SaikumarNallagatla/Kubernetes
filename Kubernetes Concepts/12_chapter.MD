# 📊 Kubernetes Monitoring with Prometheus & Grafana

## 🔧 Overview

Monitoring Kubernetes clusters is essential for understanding the performance, health, and usage of resources. **Prometheus** is used for collecting metrics (like CPU, memory), while **Grafana** is used for visualizing them. For logs, we use tools like **Grafana Loki**.

---

## 📦 How to Set Up Prometheus and Grafana in Kubernetes

### ✅ 1. Install Using Helm

We use the official `kube-prometheus-stack` Helm chart, which includes:

* Prometheus
* Grafana
* Alertmanager
* kube-state-metrics
* node-exporter

### ➤ Helm Commands:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

---

## 🔗 How Prometheus Integrates with Kubernetes

Prometheus uses Kubernetes API and `kube-state-metrics` to collect metrics. It auto-discovers Pods, Nodes, Deployments, etc.

To scrape custom app metrics:

* Define `ServiceMonitor` or `PodMonitor`
* Annotate service with `prometheus.io/scrape` and port

### 🛠 Example ServiceMonitor

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: my-app-monitor
  labels:
    release: prometheus
spec:
  selector:
    matchLabels:
      app: my-app
  endpoints:
  - port: http
```

---

## 🖥️ Accessing Grafana Dashboard

### ➤ Port Forward Grafana:

```bash
kubectl port-forward svc/prometheus-grafana 3000:80 -n monitoring
```

Open in browser: `http://localhost:3000`

* Username: `admin`
* Password:

```bash
kubectl get secret --namespace monitoring prometheus-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Grafana will already have Prometheus as a data source.

---

## ❗ Important Notes on Logs vs Metrics

| Tool             | Purpose                 |
| ---------------- | ----------------------- |
| **Prometheus**   | Metrics (CPU, RAM, etc) |
| **Grafana**      | Dashboards              |
| **Grafana Loki** | Logs                    |
| **Fluent Bit**   | Log forwarding          |

Prometheus **does NOT collect logs**.
To collect logs:

* Use FluentBit → sends logs to Loki
* View logs in Grafana

---

## 🤔 Common Doubts and Clarifications

### 🔹 "Prometheus logs chusthada?"

No. Prometheus only collects metrics. Logs need Loki.

### 🔹 "Grafana lo logs ela vasthai?"

Grafana connects to Loki as a log data source. Use Fluent Bit to send logs.

### 🔹 "Eppudu Prometheus vadali?"

Use Prometheus:

* To monitor resource usage
* To trigger alerts (ex: CPU > 90%)
* To visualize in Grafana

---

## 👨‍💻 DevOps Engineer Point of View

* Metrics → Prometheus
* Logs → Loki
* Dashboards → Grafana
* Alerts → Alertmanager

Keep monitoring setup centralized to manage complex microservices and improve SRE practices.

---

## 📌 Summary

* Prometheus + Grafana → Metrics + Dashboards
* Loki + FluentBit → Logs
* Helm makes installation easier
* DevOps engineers should know how to monitor both metrics and logs properly

---

Shall we move to the next topic (Logging Stack: Loki, Fluentd/Fluent Bit)?
