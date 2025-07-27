# Kubernetes: ServiceAccounts + RoleBinding (Pod-level Permissions)

## 🌐 What is a ServiceAccount?
A **ServiceAccount** in Kubernetes provides an identity for processes (usually Pods) running inside the cluster. It is used when pods need to interact with the Kubernetes API.

### Default Behavior:
- Every pod gets a default `default` service account (unless specified).
- If a pod doesn't need access to K8s API, no need to change it.

---

## 🔑 Why Do We Need ServiceAccounts?
- You want a pod to **list Secrets**, **access ConfigMaps**, or **manage Deployments**, etc.
- You want to follow **least privilege** principles (DevOps best practice).
- Used in **CI/CD** jobs, **GitOps** tools, and **custom controllers**.

---

## ⚖️ What is RoleBinding?
To give permissions to a ServiceAccount, we must bind it to a **Role** or **ClusterRole** using a **RoleBinding** or **ClusterRoleBinding**.

### Types of Bindings:
| Binding Type       | Scope      | Description                                |
|--------------------|------------|--------------------------------------------|
| Role + RoleBinding | Namespace  | Limited to one namespace                   |
| ClusterRoleBinding | Cluster    | Can access across all namespaces (be careful!) |

---

## 🔧 Real Use Case:
Let's say you have a **logging pod** that needs to read logs from other pods:
- Create a ServiceAccount for the logging pod.
- Create a Role to allow `get`, `list` pods.
- Bind them using a RoleBinding.

---

## ✍️ YAML Examples

### 1. Create a ServiceAccount
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: log-reader
  namespace: dev-team
```

### 2. Create a Role (namespace scoped)
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev-team
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

### 3. Create RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev-team
subjects:
- kind: ServiceAccount
  name: log-reader
  namespace: dev-team
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 🔍 Common Doubts Cleared

### Q. Can one ServiceAccount be used in multiple pods?
**Yes**, you can attach the same ServiceAccount to many pods.

### Q. Can one Role be used with multiple RoleBindings?
**Yes**, a Role can be reused.

### Q. What happens if I don't define any RoleBinding?
The ServiceAccount will have **no special permissions**, just default limited access.

---

## 🔮 DevOps Best Practices
- ✅ Always define **custom ServiceAccounts** for apps needing API access.
- ✅ Use **RoleBinding** for namespace scope (least privilege).
- ⚠️ Avoid ClusterRoleBinding unless absolutely required (like for GitOps).

---

## 🤝 Interview Tips
- Explain why RBAC is important.
- Differentiate Role vs ClusterRole.
- Real example: CI/CD tools (like ArgoCD) use custom ServiceAccounts + ClusterRoles to deploy apps.

---

Ready for next topic: **Centralized Logging (EFK / Loki)**?
# Centralized Logging in Kubernetes (EFK & Loki Stack)

## 📘 Overview

Centralized logging in Kubernetes helps manage, store, and analyze logs from multiple pods and nodes in one place. This is essential for debugging, monitoring, and alerting.

---

## 🔧 Problem with Default `kubectl logs`

* Only shows logs of running pods.
* Logs lost if pod restarts or is deleted.
* No central view of multiple services.
* No filtering, alerting, or long-term storage.

---

## 🔍 Centralized Logging Solutions

Two common stacks:

### 1️⃣ EFK Stack

* **Elasticsearch**: Stores and indexes log data.
* **Fluentd**: Collects logs from all containers and nodes.
* **Kibana**: Visualizes and searches logs.

#### 💡 Use Case:

* Good for production environments.
* Powerful search and filtering.

### 2️⃣ Loki Stack

* **Loki**: Lightweight, log-optimized storage (labels like Prometheus).
* **Promtail**: Collects logs from pods.
* **Grafana**: UI for querying logs.

#### 💡 Use Case:

* Easy integration with existing Grafana dashboards.
* Less resource usage, suitable for dev/test environments.

---

## ⚙️ Sample Architecture Diagram

```
       +-----------+         +--------------+          +-----------+
       |   Pods    |  ==>    | Fluentd/Promtail |  ==>   |  Storage  |
       |           |         |   (Collector)   |        | (Loki/ES) |
       +-----------+         +--------------+          +-----------+
                                            
                                             ==> Grafana/Kibana
```

---

## 🧪 Setup Resources

### 🔸 EFK Setup YAMLs

* Fluentd DaemonSet
* Elasticsearch StatefulSet
* Kibana Deployment

### 🔹 Loki Setup YAMLs

* Promtail DaemonSet
* Loki Deployment
* Grafana Deployment (with Loki data source)

---

## 🔐 DevOps Best Practices

* Avoid logging secrets or credentials.
* Set up log rotation & retention.
* Secure Fluentd/Promtail communication.
* Use resource limits for logging components.

---

## 🎯 Interview Questions

* Difference between EFK and Loki.
* What is Fluentd and why is it used?
* How do you prevent log flooding?
* How do you handle sensitive data in logs?
* How do logs differ from metrics?

---

## ✅ Summary

| Feature           | EFK Stack     | Loki Stack         |
| ----------------- | ------------- | ------------------ |
| Storage Backend   | Elasticsearch | Loki (label-based) |
| Collector         | Fluentd       | Promtail           |
| UI                | Kibana        | Grafana            |
| Resource Usage    | High          | Low                |
| Ideal Environment | Production    | Dev/Test           |

---

Next: Add YAML files and Grafana dashboards for both stacks.
