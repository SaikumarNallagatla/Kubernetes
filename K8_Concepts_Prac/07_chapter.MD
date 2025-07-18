```md
# 🧱 Kubernetes Step 7: StatefulSets with Persistent Identity & Storage

---

## 📚 Overview

A **StatefulSet** is a powerful controller in Kubernetes designed for **stateful applications** that require:

- Stable **network identity** (e.g., `mongo-db-0`)
- Dedicated, persistent **volume per Pod**
- Ordered **start-up and shutdown**
- Reliable **data isolation**

This contrasts with Deployments, which assume every Pod is interchangeable — perfect for **stateless microservices**, but **not for databases** or any service that remembers things.

---

## 🧠 Real-World DevOps Use Case: E-commerce Platform "ShopVerse"

In a production environment, `ShopVerse` uses:

| Component       | Purpose                           | K8s Strategy             |
|-----------------|-----------------------------------|--------------------------|
| MongoDB         | Product catalog & user profiles   | StatefulSet + PVC        |
| Redis           | Cart sessions & flash deals       | StatefulSet + PVC        |
| Frontend API    | Stateless service                 | Deployment + Service     |
| Payment Gateway | Tokenized payments via Stripe     | Secure Deployment + ConfigMap + Secret |

---

## 🔧 Hands-On: MongoDB via StatefulSet with Persistent Storage

### ✅ Step 1: Create Static PersistentVolumes

In sandbox environments (no dynamic StorageClass), manually create hostPath-backed PVs:

```yaml
# mongo-pv-0.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongo-pv-0
spec:
  capacity:
    storage: 1Gi
  accessModes: [ReadWriteOnce]
  hostPath:
    path: /data/mongo-0
```

Repeat for `mongo-pv-1` and `mongo-pv-2` with paths `/data/mongo-1` and `/data/mongo-2`.

```bash
kubectl apply -f mongo-pv-0.yaml
kubectl apply -f mongo-pv-1.yaml
kubectl apply -f mongo-pv-2.yaml
```

---

### ✅ Step 2: Deploy StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongo-db
spec:
  serviceName: "mongo"
  replicas: 3
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongodb
        image: mongo:latest
        ports:
        - containerPort: 27017
        volumeMounts:
        - name: mongo-data
          mountPath: /data/db
  volumeClaimTemplates:
  - metadata:
      name: mongo-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 1Gi
      storageClassName: ""  # Ensures manual PV binding in sandbox
```

```bash
kubectl apply -f mongo-statefulset.yaml
kubectl get pods -l app=mongo
kubectl get pvc
```

✅ You should see Pods: `mongo-db-0`, `mongo-db-1`, `mongo-db-2`  
Each with PVCs: `mongo-data-mongo-db-0`, etc.

---

### 🖼️ Architecture Diagram

```plaintext
+----------------------------+
|  StatefulSet: mongo-db     |
+-------------+--------------+
              ↓
  +---------------------+      +---------------------+      +---------------------+
  | Pod: mongo-db-0     |      | Pod: mongo-db-1     |      | Pod: mongo-db-2     |
  | PVC: mongo-data-0   |      | PVC: mongo-data-1   |      | PVC: mongo-data-2   |
  | DNS: mongo-db-0.mongo|     | DNS: mongo-db-1.mongo|     | DNS: mongo-db-2.mongo|
  +---------------------+      +---------------------+      +---------------------+
```

---

## 🧯 Troubleshooting Cheatsheet

| Issue                         | Diagnosis                                 | Fix                               |
|-------------------------------|--------------------------------------------|-----------------------------------|
| PVC stuck in `Pending`        | No matching PV available                  | Manually create HostPath PVs      |
| Pod stuck in `Pending`        | PVC unbound                               | Create PV, check accessModes      |
| StatefulSet Pods not starting | Image pull failure or storage mismatch    | Verify image and PVC setup        |
| PV not bound                  | Name mismatch or mode incompatibility     | Match PVC specs exactly           |

---

## 🚀 DevOps Principles Applied

| Principle           | Implementation                                  |
|---------------------|--------------------------------------------------|
| Declarative Infra   | YAML config for StatefulSets + PVC templates     |
| Persistent Storage  | Dedicated volume per Pod                         |
| Resilience          | Pod auto-restart with same name + data           |
| Observability       | Per-Pod logs and monitoring                      |
| Backup Strategy     | PVC per Pod → easier snapshot automation         |

---

## 💬 Interview Questions

### Q1: Why use StatefulSet over Deployment for MongoDB?
> MongoDB requires persistent data and stable pod identity; Deployment pods are ephemeral.

### Q2: What does `volumeClaimTemplates` do?
> It auto-generates a unique PVC per Pod (e.g., `mongo-data-0`, `mongo-data-1`).

### Q3: How does DNS work in StatefulSets?
> Each Pod gets a predictable DNS name: `pod-0.svc-name.namespace.svc.cluster.local`

### Q4: What happens if `mongo-db-1` crashes?
> Kubernetes re-creates it with the same name and mounts the original PVC.

---

## 👨‍💻 Author Notes

- **Name**: Bujji  
- **Platform**: Kubernetes O'Reilly Sandbox  
- **Skills Applied**: Stateful identity, volume binding, DevOps fault tolerance  
- **Use Case Simulated**: MongoDB in an e-commerce platform  
- **Next Step**: Step 8 — Kubernetes Jobs & CronJobs for automation tasks

---

Ready to roll into background task orchestration via **Jobs & CronJobs** — the heartbeat of CI/CD, backups, and data sync tasks? Say “Start Step 8” and I’ll trigger your automation engineer mode 💼📆⚙️
```