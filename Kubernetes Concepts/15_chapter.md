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
