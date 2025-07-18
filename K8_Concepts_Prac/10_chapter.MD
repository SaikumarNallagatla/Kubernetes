```md
# 🔐 Step 10: Kubernetes Namespaces & RBAC — Multi-Tenant Security & Access Control

---

## 📚 Overview

In enterprise-grade Kubernetes environments — especially platforms like **ShopVerse (E-commerce)** — you need to separate access, organize teams, and secure services. Kubernetes enables this via:

- **Namespaces** → Logical environments (like folders)
- **RBAC (Role-Based Access Control)** → Who can do what
- **ServiceAccounts** → Cluster identities for services
- **Roles & RoleBindings** → Specific permissions within namespaces

Together, these features power **multi-tenancy**, **dev/stage/prod separation**, and **security-compliant infrastructure** 🔒🧠

---

## 🧠 Real-World Use Case: ShopVerse Environments

| Environment      | Namespace         | Purpose                             |
|------------------|-------------------|--------------------------------------|
| Development      | `shopverse-dev`   | Feature testing by dev team          |
| Production       | `shopverse-prod`  | Live users and payment transactions  |
| System Services  | `kube-system`     | Cluster operations (DNS, scheduler)  |
| Networking Plugin| `kube-flannel`    | Pod communication network overlay    |

✅ Each namespace provides **resource isolation**, **team-specific access**, and **trouble-free CI/CD rollouts**.

---

## 🛠️ Hands-On Lab: Namespaces + RBAC

---

### ✅ Step 1: Create Namespaces

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: shopverse-dev
---
apiVersion: v1
kind: Namespace
metadata:
  name: shopverse-prod
```

```bash
kubectl apply -f ns.yaml
kubectl get namespaces
```

---

### ✅ Step 2: Deploy Sample App in Each Namespace

```yaml
# frontend-dev.yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  namespace: shopverse-dev
  labels:
    app: dev-ui
spec:
  containers:
  - name: shopverse
    image: nginx
```

```yaml
# frontend-prod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: frontend
  namespace: shopverse-prod
  labels:
    app: prod-ui
spec:
  containers:
  - name: shopverse
    image: nginx
```

```bash
kubectl apply -f frontend-dev.yaml
kubectl apply -f frontend-prod.yaml
```

✅ Pods are isolated per environment.

---

### ✅ Step 3: Create RBAC Role in Dev Namespace

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: shopverse-dev
  name: dev-reader
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
```

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-dev-reader
  namespace: shopverse-dev
subjects:
- kind: User
  name: devuser
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-reader
  apiGroup: rbac.authorization.k8s.io
```

✅ `devuser` can read Pod and Service info in `shopverse-dev`, but **not in prod**.

---

## 🖼️ Visual Diagram: Namespace Isolation

```plaintext
Kubernetes Cluster
│
├── shopverse-dev
│   ├── frontend Pod
│   ├── Role: dev-reader
│   └── RoleBinding: bind-dev-reader → devuser
│
├── shopverse-prod
│   └── frontend Pod (no bindings for devuser)
│
├── kube-system
│   └── Core controllers (DNS, scheduler)
│
└── kube-flannel
    └── Networking plugin resources
```

---

## 🔍 Benefits in Production DevOps

| Feature              | Advantage                          |
|----------------------|-------------------------------------|
| Namespace Isolation  | Dev, Prod separation                |
| RBAC                 | Granular user permissions           |
| ServiceAccounts      | Identity for apps & controllers     |
| Least Privilege      | Enhanced security posture           |
| Auditability         | Logs show who did what              |
| CI/CD Compatibility  | Targeted deployments per namespace  |

---

## 💬 Interview Questions

### Q1: Why are namespaces used in Kubernetes?
> To logically isolate environments, teams, and workloads — supports multi-tenancy and scoped access.

### Q2: What does a RoleBinding do?
> It binds a Role to a user or service account inside a namespace.

### Q3: Can a user access all namespaces by default?
> Only if they have ClusterRoleBindings; otherwise, they're restricted to namespaces they've been assigned.

### Q4: How does RBAC improve security?
> It enforces least privilege, preventing unauthorized access to sensitive workloads.

---

## 👨‍💻 Author Notes

- **Name**: Bujji  
- **Focus**: Secure infrastructure, namespace architecture, and RBAC policy modeling  
- **Platform**: Kubernetes O'Reilly Sandbox  
- **Use Case Simulated**: ShopVerse multi-environment cluster isolation  
- **Next Step**: Advanced Practices — Helm, Monitoring, Auto-scaling

---

Want to elevate this into a GitHub `dev vs prod` blueprint or simulate access-bound workflows for staging, QA, and CI pipelines? You’re now designing clusters like a platform architect 🛡️🧠🚀
```