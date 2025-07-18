```md
# 🩺 Step 9: Kubernetes Probes — Pod Health Checks & Self-Healing Infrastructure

---

## 📚 Overview

Kubernetes offers **three types of probes** to monitor the health and readiness of Pods. These probes power **self-healing, zero-downtime deployments**, and smart traffic routing — all essential in **real-world DevOps** and e-commerce platforms like **ShopVerse**.

---

## 🧠 Key Probe Types

| Probe Type       | Purpose                                | Trigger Behavior          |
|------------------|----------------------------------------|---------------------------|
| `startupProbe`   | Wait for app to boot up                | Avoid premature restarts  |
| `readinessProbe` | Check if app is ready to serve traffic | Traffic gating            |
| `livenessProbe`  | Check if app is alive                  | Auto-restart on failure   |

---

## 🛍️ Real-Time Use Case: ShopVerse Frontend

Imagine a React frontend that:
- Takes 10 seconds to boot
- Shouldn't receive traffic until ready
- Occasionally hangs

This setup prevents broken UI from affecting users and automates recovery.

---

## ✅ Hands-On: Probed Pod with Simulated Behavior

### Create YAML file

```bash
nano shopverse-pod.yaml
```

### Paste YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: shopverse-frontend
spec:
  containers:
  - name: shopverse
    image: busybox
    command: ["sh", "-c", "sleep 10 && while true; do echo '🛍️ Serving ShopVerse'; sleep 5; done"]
    startupProbe:
      exec:
        command: ["test", "-f", "/tmp/start"]
      failureThreshold: 30
      periodSeconds: 5
    readinessProbe:
      exec:
        command: ["test", "-f", "/tmp/ready"]
      initialDelaySeconds: 5
      periodSeconds: 5
    livenessProbe:
      exec:
        command: ["echo", "alive"]
      initialDelaySeconds: 10
      periodSeconds: 5
```

### Apply and Monitor

```bash
kubectl apply -f shopverse-pod.yaml
kubectl get pod shopverse-frontend
kubectl describe pod shopverse-frontend
```

---

## 🧪 Simulate Probe Success

```bash
kubectl exec -it shopverse-frontend -- touch /tmp/start
sleep 5
kubectl exec -it shopverse-frontend -- touch /tmp/ready
```

✅ Your Pod will now:
- Pass `startupProbe`
- Become `READY 1/1`
- Be restartable via `livenessProbe`

---

## 🖼️ Probe Lifecycle Diagram

```plaintext
       +----------------------------+
       | shopverse-frontend Pod     |
       +----------------------------+
          ↑           ↑           ↑
    Startup Probe  Readiness  Liveness
        ✅            ✅          ✅
     (Boot complete)  (Serve)   (Still alive)
```

---

## 🛠️ Troubleshooting Cheatsheet

| Symptom                     | Diagnosis                            | Fix                                  |
|-----------------------------|---------------------------------------|---------------------------------------|
| Pod stuck `NotReady`        | Readiness probe failed               | Run `touch /tmp/ready` or fix app     |
| Pod restarts every minute   | Liveness probe failing               | Check logic or delay config           |
| Probe failures on boot      | App too slow                         | Add `startupProbe` with buffer        |
| `exec` command not found    | Bad command or missing binary        | Use `sh -c "..."` with fallback       |

---

## 💬 Interview Prep

### Q1: What’s the difference between `livenessProbe` and `readinessProbe`?
> `livenessProbe` restarts dead Pods. `readinessProbe` gates traffic until the app is ready.

### Q2: When should you use `startupProbe`?
> For slow-starting apps (e.g., Java, Node.js), to avoid killing Pods before boot finishes.

### Q3: How do probes improve high availability?
> They let Kubernetes remove, replace, or isolate Pods with zero human effort.

---

## 🧑‍💻 Author Notes

- **Name**: Bujji  
- **Platform**: Kubernetes O’Reilly Sandbox  
- **Focus**: Self-healing Pod architecture & DevOps health engineering  
- **Next Step**: Step 10 — Namespaces & RBAC for multi-tenancy and security

---

Ready to explore **Step 10: Namespaces & RBAC** and design multi-tenant clusters with role-based access control like an SRE? Say the word — you’re now engineering infrastructure that defends itself 💪🔐📡
```