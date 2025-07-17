```md
# 🔐 Step 5: ConfigMaps & Secrets in Kubernetes — DevOps, Real-World Examples, Interview Qs & Troubleshooting

## 📚 Table of Contents

1. [Overview](#overview)
2. [Core Definitions](#core-definitions)
3. [Simple Analogy — The Bakery App 🍰](#simple-analogy--the-bakery-app-)
4. [Hands-On Demo](#hands-on-demo)
5. [Real-World Industry Examples](#real-world-industry-examples)
6. [Visual Architecture Diagram](#visual-architecture-diagram)
7. [Troubleshooting Cheatsheet](#troubleshooting-cheatsheet)
8. [DevOps Principles Applied](#devops-principles-applied)
9. [Interview Questions](#interview-questions)

---

## 🧠 Overview

ConfigMaps and Secrets allow Kubernetes workloads to be dynamic and secure by separating configuration from container logic. This enables safe deployment practices and environment-specific customizations across dev, staging, and production.

> 🔑 ConfigMaps handle general settings, while Secrets manage sensitive credentials securely.

---

## 📘 Core Definitions

| Concept     | Definition                                                                 |
|-------------|-----------------------------------------------------------------------------|
| **ConfigMap** | Kubernetes object for non-sensitive configuration data (theme, messages) |
| **Secret**    | Kubernetes object for sensitive, encrypted data (API keys, passwords)    |
| **Key-Value** | Both store data as `key: value` pairs accessible to Pods                 |
| **Injection** | Data made available via environment variables or mounted files           |
| **Encryption**| Secrets are base64 encoded & optionally encrypted at rest                |

---

## 🍰 Simple Analogy — The Bakery App

Imagine a website called **SweetStack**:

- ConfigMap holds:
  - `GREETING="Welcome to SweetStack!"`
  - `THEME="chocolate"`

- Secret holds:
  - `DB_PASSWORD="BakingRocks@2024"`
  - `API_TOKEN="sk_live_ABC123"`

```plaintext
ConfigMap = Public recipe card on the counter  
Secret    = Locked safe for ingredients and keys
```

---

## 🛠️ Hands-On Demo: Injecting ConfigMap & Secret

### ✅ Create ConfigMap

```bash
kubectl create configmap app-config --from-literal=GREETING="Hello Bujji!"
```

### ✅ Create Secret

```bash
kubectl create secret generic app-secret --from-literal=API_TOKEN="s3cr3t-t0k3n-xyz"
```

### ✅ Sample Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: config-demo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: config-demo
  template:
    metadata:
      labels:
        app: config-demo
    spec:
      containers:
      - name: demo-container
        image: busybox
        command: ["sh", "-c", "echo $GREETING && echo $API_TOKEN && sleep 3600"]
        env:
        - name: GREETING
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: GREETING
        - name: API_TOKEN
          valueFrom:
            secretKeyRef:
              name: app-secret
              key: API_TOKEN
```

### ✅ Verify Output

```bash
kubectl exec -it <pod-name> -- sh
echo $GREETING         # "Hello Bujji!"
echo $API_TOKEN        # "s3cr3t-t0k3n-xyz"
```

---

## 🏭 Real-World Industry Examples

| Company      | Use Case                                          | Resource Type |
|--------------|--------------------------------------------------|---------------|
| Stripe       | Secure token authentication                     | Secret        |
| Netflix      | Feature flags for AB testing                    | ConfigMap     |
| PayPal       | TLS certificates for encrypted traffic          | Secret        |
| GitHub CI/CD | Repo URL and environment flags                  | ConfigMap     |
| Healthcare   | Patient data encryption key                     | Secret        |
| E-commerce   | Color theme, currency toggle                    | ConfigMap     |

---

## 🖼️ Visual Architecture Diagram

```plaintext
+------------------+
|     ConfigMap     |                +----------------+
| GREETING=Hello!   |<-------+       |    Secret      |
| THEME=choco       |        |       | API_TOKEN=**** |
+------------------+        |       +----------------+
                            |
                            v
+-------------------------------+
|         Kubernetes Pod        |
|-------------------------------|
| ENV: GREETING, API_TOKEN      |
| Container: demo-container     |
+-------------------------------+
```

---

## 🛠️ Troubleshooting Cheatsheet

| Issue                           | Diagnosis                        | Fix                                    |
|----------------------------------|----------------------------------|----------------------------------------|
| Pod won’t start                 | `kubectl describe pod`           | ConfigMap/Secret missing or misnamed  |
| `$GREETING` is blank            | Inside Pod: `echo $GREETING`     | Key typo or value not injected        |
| Secret not decoded              | Base64 encoding error            | Verify `kubectl describe secret`      |
| Changes not reflected           | Pod not restarted                | Redeploy Deployment or rollout restart|

---

## 🚀 DevOps Principles Applied

| Principle       | Implementation                                |
|-----------------|-----------------------------------------------|
| **Security**     | Secrets encrypted + RBAC access               |
| **Portability**  | Images reused across environments             |
| **Automation**   | Injected via CI/CD at runtime                 |
| **Separation of concerns** | Config outside image               |
| **Observability**| Config-driven logging verbosity               |

---

## 💬 Interview Questions

### Q1: When would you use a Secret instead of a ConfigMap?
> When storing sensitive data like API keys, passwords, TLS certs.

### Q2: How are Secrets stored in Kubernetes?
> Base64-encoded in etcd; optionally encrypted at rest.

### Q3: Can ConfigMap and Secret be mounted as files?
> Yes, using volumeMounts and volumes in Pod spec.

### Q4: What happens if a Secret is missing during Pod creation?
> Pod enters error state; check `describe` for failure message.

### Q5: How would you rotate an API token stored in a Secret?
> Update Secret object → trigger Pod restart or rollout

---

## 👨‍💻 Author

- **Name**: Bujji  
- **Focus**: Cloud-native DevOps, Kubernetes, GitHub documentation  
- **Platform**: O'Reilly Kubernetes Sandbox  
- **Next Step**: Step 6 — Volumes, Persistent Storage & Stateful Workloads

---

