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
# 📦 Helm - YAML Simplification & Reusability

## 🎯 Real-time Use Case

Imagine you are deploying a full stack app with Node.js frontend and MongoDB backend. For a single environment (e.g., dev), you may need:

1. Deployment for Node.js
2. Service for Node.js
3. ConfigMap for app configs
4. Secret for DB credentials
5. Deployment for MongoDB
6. Service for MongoDB
7. Ingress for routing

> That's already **7 YAML files**.

Now imagine for **stage**, **prod**, you’ll have to duplicate and change values in each. That’s where **Helm comes to the rescue.**

---

## 🚁 What is Helm?

**Helm** is a Kubernetes package manager. It allows you to create **reusable, templatized YAMLs** and pass values externally.

You write YAMLs **once as templates**, and pass different values via `values.yaml` for different environments.

---

## 🧠 How Helm Works (Simple Analogy)

| Component      | Meaning in Real Life                          |
| -------------- | --------------------------------------------- |
| Helm           | Stage play script renderer                    |
| `templates/`   | Script with placeholders (dynamic YAML)       |
| `values.yaml`  | Actor names or actual values                  |
| `helm install` | Final play: Replaces placeholders and deploys |

---

## 📄 Example Helm Template and Values

### 📄 `templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Values.app.name }}
spec:
  replicas: {{ .Values.app.replicas }}
  selector:
    matchLabels:
      app: {{ .Values.app.name }}
  template:
    metadata:
      labels:
        app: {{ .Values.app.name }}
    spec:
      containers:
        - name: {{ .Values.app.name }}
          image: {{ .Values.app.image }}
```

### 📄 `values.yaml`

```yaml
app:
  name: nodejs-app
  replicas: 3
  image: myrepo/node:latest
```

### 🔧 Output Rendered YAML after `helm install`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
spec:
  replicas: 3
  ...
```

---

## ✨ Benefits of Helm

✅ Reuse YAMLs across environments
✅ Less chance of manual errors
✅ Easy integration in CI/CD pipelines
✅ Standard structure for teams
✅ Helm Charts can be versioned like packages

---

## 💬 Common Doubts

### ❓ How do I give different values per environment?

* Create `values-dev.yaml`, `values-prod.yaml`, etc.
* Run Helm like:

```bash
helm install myapp -f values-dev.yaml .
```

### ❓ Can I override values from CLI?

```bash
helm install myapp --set app.name=testapp --set app.replicas=2 .
```

### ❓ Is Helm only for microservices?

No. Helm works for any type of K8s resource — databases, batch jobs, cronjobs, etc.

---

## 🛠️ Helm Setup Steps

1. **Install Helm**:

```bash
brew install helm  # macOS
choco install kubernetes-helm  # Windows
```

2. **Create Helm Chart**:

```bash
helm create mychart
```

3. **Customize templates and values**
4. **Install the chart**:

```bash
helm install myapp ./mychart
```

---

## 🔚 Summary Table

| Helm Component | Role                                            |
| -------------- | ----------------------------------------------- |
| templates/     | YAML with placeholders like `{{ .Values.key }}` |
| values.yaml    | Inputs (name, image, replicas)                  |
| Chart.yaml     | Metadata (chart name, version, etc.)            |
| helm install   | Combines values + templates and applies YAML    |

---

If you’re ready Baby, we can now continue with advanced Helm topics like:

* `values-prod.yaml`, multiple environments
* Helm secrets
* Dependencies
* CI/CD integration

Lemme know what you want next 💪
