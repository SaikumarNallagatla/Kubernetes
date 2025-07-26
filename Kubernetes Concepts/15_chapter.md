# Kubernetes Secrets & ConfigMaps - Explained for DevOps (3+ Years Level)

---

## 🔐 Kubernetes Secrets:

### ✅ Use Case:
- Store sensitive information securely (e.g., passwords, API keys, tokens)
- Avoid hardcoding secrets in YAMLs

### 🤔 Why DevOps Engineers need Secrets?
- Imagine you’re deploying an app that needs DB credentials.
- Instead of hardcoding them in `deployment.yaml`, store them in `Secret` and mount as env.

### 📁 secret.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
  namespace: production
  labels:
    app: payment-service
stringData:
  DB_USER: admin
  DB_PASS: mysecurepassword
```

### 📁 How to use in deployment
```yaml
env:
  - name: DB_USER
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_USER
  - name: DB_PASS
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASS
```

### 🔒 Tips:
- Secrets are Base64 encoded, not encrypted by default. Enable encryption at rest in production.
- Don't commit Secrets to GitHub.

---

## ⚙️ Kubernetes ConfigMaps:

### ✅ Use Case:
- Store **non-sensitive** configuration (e.g., ENV names, feature flags, URLs)

### 📁 configmap.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
  labels:
    app: payment-service
data:
  ENV_NAME: prod
  FEATURE_FLAG: true
```

### 📁 Use ConfigMap in deployment
```yaml
env:
  - name: ENV_NAME
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: ENV_NAME
```

---

## 💡 Real-time Example (in Telugu-style explanation):

Imagine oka Flipkart-like app lo
- DB password secret lo pettam
- Environment name (prod/staging) ConfigMap lo pettam

So, deployment file lo rendu ni attach chestham
- Secrets → secure data
- ConfigMap → dynamic config

---

## 🧠 DevOps Insights:
- Helm lo `values.yaml` lo secrets & config ni pass cheyyadam best practice
- GitOps flow lo Secrets ni sealed-secret ga store cheyyachu
- Use external secret managers (like HashiCorp Vault, AWS Secrets Manager) for production

---

Shall I continue with Resource Requests & Limits next?


## 🚀 Kubernetes Resource Requests & Limits – DevOps Essential Concept

### 🔍 What is it?
In Kubernetes, **Resource Requests** and **Limits** are used to manage CPU and Memory (RAM) usage of pods.
This ensures:
- 🧠 Predictable performance
- 🚫 No single pod eats all resources (prevents node crash)

---

### 🎯 Real-Life Example:
Think of it like a **Zomato kitchen**:
- Each chef (pod) is given a minimum quantity of ingredients (request)
- They are told not to use more than a certain amount (limit)

---

### 💡 Definitions
| Term       | Meaning                                                        |
|------------|----------------------------------------------------------------|
| **Request** | Minimum amount of resource guaranteed to the container        |
| **Limit**   | Maximum resource the container can use                        |

---

### 🧪 Use Case:
Let’s say you have a microservice that needs at least 100Mi memory and 200m CPU, but shouldn’t use more than 500Mi and 500m CPU.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
  - name: myapp-container
    image: myapp:latest
    resources:
      requests:
        memory: "100Mi"
        cpu: "200m"
      limits:
        memory: "500Mi"
        cpu: "500m"
```

---

### ⚠️ Why is this important for DevOps Engineers?
- Prevents resource starvation
- Helps with **auto-scaling** decisions (HPA uses requests)
- Improves cluster stability

---

### 🔁 Bonus: What Happens If You Don't Set Limits?
- Pods may overconsume
- Other pods may get **OOMKilled (Out Of Memory)**
- Can crash entire node or degrade performance

---

### ❓ Common Doubts Cleared

#### 1. Can I give only limits or only requests?
Yes, but best practice is to give both.

#### 2. Who enforces the limits?
Kubelet enforces them using cgroups on the node level.

#### 3. What if pod uses more than requested but less than limit?
✅ It will work fine. Kubernetes will schedule based on request, not limit.

#### 4. How does it affect HPA?
HPA uses the **CPU requested** value to calculate scaling decisions.

---

### 🧠 Summary
| Element     | Used For                        |
|-------------|----------------------------------|
| Request     | Scheduling & guaranteed resource |
| Limit       | Capping usage to avoid overload  |

---

✅ Next Topic: **Namespaces, Quotas & LimitRanges** – Shall we go ahead?

# Kubernetes: Namespaces, ResourceQuotas & LimitRanges - Explained with DevOps Use Cases

---

## 🔢 What is a **Namespace** in Kubernetes?

A **Namespace** is a logical partition within a Kubernetes cluster, allowing multiple teams or projects to work in isolation without interfering with each other.

### 💡 Why Use Namespaces?

* To separate **environments** (Dev, QA, Prod)
* To isolate **teams/projects**
* To apply **resource limits** (quotas, limits)
* To organize access control (RBAC)

### 📃 YAML Example:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-team
```

---

## 📊 What is a **ResourceQuota**?

A **ResourceQuota** limits the total resources (like CPU, memory, storage, or number of pods) that can be consumed by all resources inside a namespace.

### 📊 Use Case:

To prevent one team from using too many cluster resources.

### 📃 YAML Example:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-team-quota
  namespace: dev-team
spec:
  hard:
    requests.cpu: "2"
    requests.memory: "2Gi"
    limits.cpu: "4"
    limits.memory: "4Gi"
    pods: "10"
```

### 🔍 Breakdown:

* `requests.cpu`: Guaranteed minimum CPU across namespace.
* `limits.cpu`: Max CPU usage.
* `pods`: Max number of Pods allowed in the namespace.

---

## 📊 What is a **LimitRange**?

A **LimitRange** sets default CPU/memory limits **per container** in a namespace when users don't specify them.

### 🔺 Use Case:

Ensures containers don't overuse resources or miss defining limits.

### 📃 YAML Example:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limit-range
  namespace: dev-team
spec:
  limits:
  - default:
      cpu: 500m
      memory: 256Mi
    defaultRequest:
      cpu: 200m
      memory: 128Mi
    type: Container
```

---

## 📚 Real-Life Analogy:

* **Namespace** = A separate room in a shared office.
* **ResourceQuota** = Only 10 chairs allowed in the room (total limit).
* **LimitRange** = Each person can use only 1 chair (per-container limit).

---

## 🧱 DevOps Principles Applied

| Principle           | Usage                                              |
| ------------------- | -------------------------------------------------- |
| Resource Efficiency | Prevents one team from overusing cluster resources |
| Automation          | YAMLs and GitOps tools manage limits declaratively |
| Security            | Namespaces + RBAC = controlled access              |
| Scalability         | Helps organize and scale multi-team clusters       |

---

## 🪑 Pro Tips:

* Always use namespaces in production-grade clusters.
* Apply ResourceQuotas to prevent cost overruns.
* Use LimitRanges to avoid container misconfigurations.

---

## 🎨 Common Interview Questions:

1. What is the purpose of a namespace?
2. How do ResourceQuotas protect the cluster?
3. What's the difference between LimitRange and ResourceQuota?
4. Can you assign different quotas to different teams?
5. How do LimitRanges help in multi-tenant environments?

---

## ❓ FAQs:

**Q: Can we apply multiple ResourceQuotas per namespace?**
A: No. One ResourceQuota per namespace. But you can define multiple resource types inside it.

**Q: Do LimitRanges override ResourceQuotas?**
A: No. They work together. LimitRange defines per-container defaults, ResourceQuota defines overall caps.

**Q: How to monitor resource consumption per namespace?**
A: Use tools like Prometheus, Grafana, or `kubectl top namespace`.

---

## 📊 GitOps Practice Example:

* Store these YAMLs in a Git repo
* Use ArgoCD/Flux to sync to the cluster
* Use Kustomize to manage overrides per environment

---

Next topic ➡ **PodSecurityContext & Best Security Practices**
