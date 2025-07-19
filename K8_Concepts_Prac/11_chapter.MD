```md
# 📦 ShopVerse Helm Deployment: Concepts, Architecture & Step-by-Step Guide

---

## 🧠 What Is Helm in Kubernetes?

**Helm** is the official **package manager for Kubernetes**. It lets you define, install, and upgrade complex Kubernetes applications using simple, reusable templates called **charts**.

| Feature       | Helm Equivalent  |
|---------------|------------------|
| Chart         | App package      |
| values.yaml   | Config file      |
| templates/    | Templated manifests |
| `helm install`| Deploy the app   |

---

## 🎯 Why Use Helm?

Without Helm:
- You manually create many YAML files for each environment
- CI/CD becomes messy with duplicate code
- Scaling or rollback is slow and error-prone

With Helm:
✅ Reuse your Kubernetes logic  
✅ Customize per environment via config  
✅ Version, upgrade, and rollback with ease  
✅ GitOps & CI/CD friendly

---

## 🛍️ Use Case: ShopVerse Frontend

We deployed **ShopVerse**, an e-commerce frontend, to two environments — dev and prod — using Helm. Both use the same chart, but different configs via `values.yaml`.

---

## 🧱 Architecture Diagram

```plaintext
Kubernetes Cluster
│
├── Namespace: shopverse-dev
│   ├── Helm Release: shopverse-dev
│   ├── Config: values.yaml
│   └── Replicas: 2 (nginx:latest)
│
├── Namespace: shopverse-prod
│   ├── Helm Release: shopverse-prod
│   ├── Config: values-prod.yaml
│   ├── Replicas: 3 (nginx:1.25.2)
│   ├── Autoscaling: Enabled
│   └── Resources: CPU/memory limits
│
└── Shared Chart: shopverse-frontend
    ├── Chart.yaml
    ├── values.yaml / values-prod.yaml
    └── templates/
```

---

## 🛠️ Step-by-Step: Dev & Prod Rollout

### ✅ Step 1: Create Helm Chart

```bash
helm create shopverse-frontend
```

Auto-generates a chart folder with:
- `Chart.yaml`
- `values.yaml`
- `templates/` containing `deployment.yaml`, `service.yaml`, etc.

---

### ✅ Step 2: Customize `values.yaml` for Dev

```yaml
replicaCount: 2
image:
  repository: nginx
  tag: latest
```

---

### ✅ Step 3: Create Dev Namespace & Install Chart

```bash
kubectl create namespace shopverse-dev
helm install shopverse-dev ./shopverse-frontend -n shopverse-dev
```

---

### ✅ Step 4: Create `values-prod.yaml` for Prod

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.25.2"
  pullPolicy: IfNotPresent

resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "500m"
    memory: "1Gi"

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 75

nameOverride: "shopverse"
fullnameOverride: "shopverse-prod-app"
```

Save as `values-prod.yaml`

---

### ✅ Step 5: Create Prod Namespace & Install Chart

```bash
kubectl create namespace shopverse-prod
helm install shopverse-prod ./shopverse-frontend -n shopverse-prod -f values-prod.yaml
```

---

### ✅ Step 6: Port Forward for Testing

```bash
kubectl port-forward <pod-name> 8080:80 -n shopverse-prod
```

Then visit:
```
http://127.0.0.1:8080
```

---

## 🔄 Step 7: Helm Lifecycle Commands

| Task         | Command Example                                      |
|--------------|------------------------------------------------------|
| Upgrade      | `helm upgrade shopverse-prod ...`                    |
| Rollback     | `helm rollback shopverse-prod 1 -n shopverse-prod`  |
| View History | `helm history shopverse-prod -n shopverse-prod`     |
| Uninstall    | `helm uninstall shopverse-prod -n shopverse-prod`   |

---

## 📦 Summary

You now have:
| Environment  | Namespace       | Helm Release     | ReplicaCount | Tag       |
|--------------|------------------|------------------|--------------|-----------|
| Dev          | shopverse-dev    | shopverse-dev    | 2            | latest    |
| Prod         | shopverse-prod   | shopverse-prod   | 3            | 1.25.2    |

✅ Helm gave you reusable infrastructure  
✅ Easy scaling, rollback, and version control  
✅ Real-world DevOps blueprint for GitHub portfolio

---

## 👨‍💻 Author: Bujji

- Structured learner & Kubernetes explorer  
- Skilled in troubleshooting & multi-env deployment  
- Passionate about sharing knowledge with GitHub-readiness

---

Let me know if you want this polished into a GitHub-flavored `README.md` with CI/CD and RBAC notes next. You're officially deploying like a platform engineer in production 🔧📦💥
```