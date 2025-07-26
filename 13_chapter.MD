
# Kubernetes RBAC (Role-Based Access Control) - Explained with Use Cases

## 🌟 What is RBAC?
RBAC (Role-Based Access Control) is a mechanism in Kubernetes to restrict and control what users or service accounts can do within a cluster.

It provides fine-grained access control by assigning **roles** to **subjects** (users, groups, or service accounts) and defining what **resources** they can access and **what actions** they can perform.

---

## 🔑 Key RBAC Objects

| Object         | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| Role           | Defines permissions within a specific namespace                            |
| ClusterRole    | Defines cluster-wide permissions or can be used within namespaces          |
| RoleBinding    | Assigns a Role to a user/service account within a namespace                |
| ClusterRoleBinding | Assigns a ClusterRole to a user/service account cluster-wide         |

---

## 🧠 Basic Flow of RBAC

```text
1. Create a Role/ClusterRole with required permissions.
2. Bind that Role to a User or Service Account using RoleBinding/ClusterRoleBinding.
3. User/Service Account gets only allowed access.
```

---

## 🔧 Use Case 1: Developer should only view Pods in `dev` namespace

### 1. Role
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

### 2. RoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
- kind: User
  name: developer1
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 🔧 Use Case 2: Grant access to list all nodes cluster-wide

### ClusterRole
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
```

### ClusterRoleBinding
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes-binding
subjects:
- kind: User
  name: admin1
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## 🤔 Common Questions & Answers

### ❓ Who creates RBAC rules?
RBAC rules are usually created by cluster admins to control access for developers, testers, CI/CD pipelines, etc.

### ❓ What happens if a user doesn’t have proper RBAC?
They’ll get an error like: `"Forbidden: User XYZ cannot get resource pods"`

### ❓ Can Role be used for multiple namespaces?
No. Use `ClusterRole` and bind it with `RoleBinding` in multiple namespaces.

---

## 📌 Real-World Example: Give Jenkins ServiceAccount access to deploy apps
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: deploy-role
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update"]
```
```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: jenkins-deploy-binding
subjects:
- kind: ServiceAccount
  name: jenkins
  namespace: cicd
roleRef:
  kind: ClusterRole
  name: deploy-role
  apiGroup: rbac.authorization.k8s.io
```

---

## ✅ Summary
- Use `Role` and `RoleBinding` for namespace-level access.
- Use `ClusterRole` and `ClusterRoleBinding` for cluster-wide access.
- Always follow least privilege principle.
- RBAC is essential for DevOps engineers for securing and managing cluster access.

---

> 🔐 Pro Tip: Always test your RBAC policies in a dev/staging cluster before applying in production!


Let’s move to the next topic when you're ready!







# 📘 Kubernetes RBAC (Role-Based Access Control)

RBAC is a Kubernetes security mechanism that allows you to define **who can access what** in your cluster.

---

## 🎯 Use Case: Give Access to a Specific Person or Service

To give someone or something (like Jenkins or Prometheus) access to specific Kubernetes resources, follow these **3 steps**.

---

## ✅ Step 1: Create a ServiceAccount

A ServiceAccount is an identity used within the cluster.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dev-user
  namespace: dev
```

👉 This creates a service account named `dev-user` in the `dev` namespace.

---

## ✅ Step 2: Create Role or ClusterRole

* Use a `Role` if access should be limited to a namespace.
* Use a `ClusterRole` for cluster-wide access.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: dev-role
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
```

👉 This `Role` allows `get`, `list`, `watch` on pods and services within the `dev` namespace.

---

## ✅ Step 3: Bind Role to ServiceAccount

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-role-binding
  namespace: dev
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: dev
roleRef:
  kind: Role
  name: dev-role
  apiGroup: rbac.authorization.k8s.io
```

👉 This binds the `dev-role` permissions to `dev-user` ServiceAccount.

---

## 🔐 How to Authenticate Using ServiceAccount

To give someone access via `kubectl`, extract the token:

```bash
kubectl -n dev get secret | grep dev-user
kubectl -n dev describe secret <secret-name>
```

👉 Give this token to the user or CI tool. They can configure `kubectl` to use it.

---

## 💡 Real-Time Example (DevOps Use Case)

You're a DevOps engineer at TCS. You need to give Jenkins pipeline access to only `dev` namespace:

* ✅ Create a ServiceAccount (`jenkins-sa`)
* ✅ Define a Role with limited permissions (only `get`, `list` pods)
* ✅ Bind the Role using RoleBinding
* ✅ Extract the token and configure it inside Jenkins.

---

## ❓ Common Doubts:

### ❓ Is Role and RoleBinding enough for full access?

No, `Role` works within a namespace. For cluster-wide resources like `nodes`, use `ClusterRole` and `ClusterRoleBinding`.

### ❓ Can I attach this to a human user?

Kubernetes doesn’t manage human identities. You typically integrate with your cloud provider IAM, OIDC, or use tokens with `kubectl` config.

---

Let’s continue to the next topic when you're ready!
