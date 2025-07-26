# Helm in Kubernetes - Complete Guide for DevOps Engineers

## 📌 What is Helm?

Helm is a **package manager** for Kubernetes. It simplifies deployment, versioning, and management of Kubernetes applications by using **charts** (predefined templates).

Think of Helm like **`apt` or `yum`** for Kubernetes.

---

## 🎯 Why Helm?

* Reuse Kubernetes configurations (templates)
* Manage complex deployments with variables
* Perform upgrades, rollbacks, and lifecycle management
* Package your entire app with all dependencies

---

## 🔧 Helm Components

| Component     | Description                                              |
| ------------- | -------------------------------------------------------- |
| **Chart**     | A Helm package. It contains templates and default values |
| **Values**    | User-defined configuration injected into templates       |
| **Templates** | YAML files with Go templating engine syntax              |
| **Release**   | A deployed chart in your cluster                         |

---

## 🗂️ Helm Chart Directory Structure

```
mychart/
├── Chart.yaml          # Metadata info (name, version)
├── values.yaml         # Default configuration values
├── templates/          # All Kubernetes YAML templates here
│   ├── deployment.yaml
│   ├── service.yaml
│   └── _helpers.tpl    # Template functions
```

---

## ✍️ Create a Simple Helm Chart

```bash
helm create myapp
cd myapp
```

It will generate default `deployment`, `service`, etc. inside `templates/`

---

## 🔁 Installing a Chart (Release)

```bash
helm install my-release ./myapp
```

To override default values:

```bash
helm install my-release ./myapp -f custom-values.yaml
```

To upgrade:

```bash
helm upgrade my-release ./myapp
```

To rollback:

```bash
helm rollback my-release 1
```

To uninstall:

```bash
helm uninstall my-release
```

---

## 🧪 Example: Deployment Template with Variable Values

`templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
          ports:
            - containerPort: {{ .Values.service.port }}
```

---

## 📘 Sample `values.yaml`

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: latest
service:
  port: 80
```

---

## 🤔 Common Doubts

### ❓ What is a Helm Release?

A release is a specific deployment instance of a chart. You can have multiple releases of the same chart with different configurations.

### ❓ Helm vs Kustomize?

| Helm                               | Kustomize                             |
| ---------------------------------- | ------------------------------------- |
| Uses templates                     | Uses overlays                         |
| More dynamic                       | More declarative                      |
| Best for reusability and packaging | Best for modifying existing manifests |

### ❓ What if I want to use Helm + GitOps?

Use **ArgoCD** or **FluxCD** to deploy Helm charts automatically from Git repos.

---

## 🧠 Real-World Use Case

You have 5 microservices (payment, order, user, auth, inventory). Instead of writing separate deployment YAMLs for each one, you can:

* Write one generic Helm chart
* Pass different values files for each microservice

---

## 🔐 Bonus: Secure Secrets in Helm

Use external secret managers (e.g., HashiCorp Vault, Sealed Secrets) and inject them through `values.yaml` securely.

---

## 🔚 Summary

* Helm simplifies Kubernetes app deployment
* Use `charts`, `values.yaml`, and templated `YAMLs`
* Ideal for microservices, versioning, and reusable infra
* Works well with GitOps (ArgoCD)

---

Next: We will cover **ConfigMaps & Secrets** with real-world use cases and secure best practices.
