```md
# 📦 Kubernetes Step 6: Persistent Volumes & Stateful Workloads

---

## 📚 Overview

In this step, we explore how Kubernetes handles data that needs to **persist across Pod restarts**. This is essential for workloads like databases, caches, file storage, and stateful services that can't afford to lose data.

Kubernetes offers:

- `Volume`: Basic storage attached to Pods
- `PersistentVolume (PV)`: Cluster-level reusable storage
- `PersistentVolumeClaim (PVC)`: Request for storage by a Pod
- `StatefulSet`: A controller that creates Pods with persistent identity and storage

---

## 🧠 Core Concepts & Definitions

| Concept            | Definition                                                                 |
|--------------------|-----------------------------------------------------------------------------|
| **Volume**         | A directory accessible by containers in a Pod                              |
| **HostPath**       | Volume that mounts a file path on the node (sandbox-safe for testing)      |
| **PersistentVolume (PV)** | Pre-provisioned storage in the cluster                           |
| **PersistentVolumeClaim (PVC)** | Pod’s request for storage                                 |
| **StatefulSet**    | Controller that assigns persistent identity and volume per Pod             |

---

## 🍰 Simple Analogy

Imagine a hotel with 5 rooms.

- **PV**: The hotel rooms
- **PVC**: A guest’s booking request
- **Pod**: The guest (application)
- **StatefulSet**: VIP guests — always get the same room number + key

---

## 🔧 Hands-On: Create & Test Persistent Storage

### ✅ 1. Create PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: demo-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/demo
```

```bash
kubectl apply -f pv.yaml
kubectl get pv
```

---

### ✅ 2. Create PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

```bash
kubectl apply -f pvc.yaml
kubectl get pvc
```

---

### ✅ 3. Create Pod That Uses the PVC

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: volume-test-pod
spec:
  containers:
  - name: app
    image: busybox
    command: ["sh", "-c", "echo BujjiWasHere > /data/persist.txt && sleep 3600"]
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: demo-pvc
```

```bash
kubectl apply -f pod.yaml
kubectl exec -it volume-test-pod -- sh
cat /data/persist.txt
```

✅ Output should be: `BujjiWasHere`

---

### ✅ 4. Delete & Reconnect Pod

Recreate a new Pod that uses the same PVC:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: reconnect-pod
spec:
  containers:
  - name: reader
    image: busybox
    command: ["sh", "-c", "cat /data/persist.txt && sleep 3600"]
    volumeMounts:
    - mountPath: "/data"
      name: storage
  volumes:
  - name: storage
    persistentVolumeClaim:
      claimName: demo-pvc
```

```bash
kubectl apply -f pod-reconnect.yaml
kubectl exec -it reconnect-pod -- sh
cat /data/persist.txt
```

✅ File persists — success!

---

## 🏭 Real-World Use Cases

| Use Case           | Explanation                                      |
|--------------------|--------------------------------------------------|
| MySQL/MongoDB      | Needs durable disk for data                      |
| Redis              | Persists cache across crash                      |
| Jenkins            | CI/CD job results stored in PVC                  |
| CMS (WordPress)    | Mounted volume for uploads and media             |

---

## 🖼️ Visual Architecture Diagram

```plaintext
+--------------------------+
| PersistentVolume (PV)    |
| HostPath: /data/demo     |
+-----------+--------------+
            ↑
+-----------+--------------+
| PersistentVolumeClaim    |
| Name: demo-pvc           |
+-----------+--------------+
            ↑
+-----------+--------------+
| Pod: volume-test-pod     |
| Mounts PVC at /data      |
+--------------------------+
```

---

## 🛠️ Troubleshooting Cheatsheet

| Problem                     | Diagnosis                           | Fix                                 |
|-----------------------------|--------------------------------------|-------------------------------------|
| Pod won't start            | `kubectl describe pod`               | PVC not bound or PV mismatch        |
| File not found             | `cat /data/persist.txt`              | File not created or incorrect mount |
| PVC stays in Pending       | `kubectl get pvc`                    | No matching PV in cluster           |
| PV stays Unbound           | `kubectl get pv`                     | AccessModes mismatch with PVC       |

---

## 🔑 Interview Questions

### Q1: What's the difference between Volume and PVC?
> Volume is Pod-specific; PVC is a storage request that binds to cluster-level PV.

### Q2: Can you use the same PVC across multiple Pods?
> Yes, if access mode is `ReadWriteMany`, else `ReadWriteOnce` allows only one.

### Q3: What is a StatefulSet and how does it relate to storage?
> A StatefulSet gives Pods stable network IDs and storage via unique PVCs.

### Q4: What is HostPath and when is it safe to use?
> HostPath mounts node directories; safe for sandbox/dev, not recommended in prod.

---

## 💡 What You Learned in This Step

- How Kubernetes handles durable storage
- The difference between Volumes, PVs, and PVCs
- How to create Pods with persistent data
- Why Stateful workloads need unique volume claims
- Hands-on YAML crafting with real runtime validation

---

## 👨‍💻 Author

- **Name**: Bujji  
- **Focus**: Cloud-native architecture, DevOps simulation, GitHub documentation  
- **Platform**: Kubernetes O’Reilly Sandbox  
- **Next Step**: **StatefulSet + VolumeClaimTemplates + Pod identity**

---

Ready to tackle stateful services like Redis, MongoDB, and Kafka with **StatefulSets and auto-volume provisioning**? Just say "StatefulSet Hands-On" and I’ll prep the next lab 🍃🧱
```