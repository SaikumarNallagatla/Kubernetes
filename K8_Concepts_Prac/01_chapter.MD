
---

```md
# 🚀 Kubernetes Basics: Hands-On Intro with Hello-World App

## 📚 Table of Contents
1. [Introduction to Kubernetes](#introduction-to-kubernetes)
2. [Docker vs Kubernetes](#docker-vs-kubernetes)
3. [Architecture Overview](#architecture-overview)
4. [Hands-On Deployment (O'Reilly Sandbox)](#hands-on-deployment-oreilly-sandbox)
5. [Commands & Checks](#commands--checks)
6. [Accessing the App](#accessing-the-app)
7. [Troubleshooting & Sandbox Tips](#troubleshooting--sandbox-tips)
8. [Production Guidelines](#production-guidelines)
9. [Interview Questions](#interview-questions)
10. [Quiz](#quiz)

---

## 🧠 Introduction to Kubernetes

**Kubernetes** (also known as **K8s**) is an open-source platform for automating deployment, scaling, and management of containerized applications.

- Created by Google, based on their internal system **Borg**
- Maintained by CNCF (Cloud Native Computing Foundation)
- “K8s” is shorthand: 8 letters between **K** and **s**

### 🔥 Why Use Kubernetes?

- Self-healing containers
- Auto-scaling based on traffic
- Declarative configuration (YAML)
- Rolling updates without downtime
- Portable across clouds and on-prem

---

## ⚔️ Docker vs Kubernetes

| Feature             | Docker Alone                 | Kubernetes                          |
|---------------------|------------------------------|--------------------------------------|
| App Packaging        | ✔️                           | ✔️ (uses Docker under the hood)     |
| Container Orchestration | ❌ Manual setup            | ✔️ Fully automated                  |
| Load Balancing      | ❌ Limited                   | ✔️ Built-in                         |
| Resilience & Healing| ❌ None                      | ✔️ Restarts crashed Pods            |
| Scaling             | ❌ Manual                    | ✔️ Horizontal auto-scaling          |

🧩 Kubernetes solves the complexity of running containers at scale, across environments.

---

## 🧱 Architecture Overview

### 🔹 Control Plane

- `API Server`: Entry point for `kubectl` & automation
- `Controller Manager`: Maintains cluster state
- `Scheduler`: Assigns Pods to nodes
- `etcd`: Stores cluster configuration
- `Cloud Controller`: Integrates cloud features

### 🔸 Worker Node

- `kubelet`: Ensures Pods are running
- `kube-proxy`: Networking inside cluster
- `Container Runtime`: Runs containers (`containerd`, `Docker`)

---

## 🧪 Hands-On Deployment (O'Reilly Sandbox)

### ✅ Step 1: Create Deployment

```bash
kubectl create deployment hello-world --image=k8s.gcr.io/echoserver:1.4
```

### ✅ Step 2: Expose as NodePort

```bash
kubectl expose deployment hello-world --type=NodePort --port=8080
```

---

## 🔍 Commands & Checks

### 🎯 Verify Deployment

```bash
kubectl get deployments
kubectl get pods
kubectl describe pod <pod-name>
```

### 🔎 Check Service & Port

```bash
kubectl get svc hello-world
```

Expected output: `NodePort` assigned (e.g., `30446`)

### 🧭 Find Node IP

```bash
kubectl get nodes -o wide
```

Look for internal IP like `172.16.28.5`

---

## 🌐 Accessing the App

**Option 1: NodePort (may be restricted in sandbox)**

```bash
curl http://<NodeIP>:30446
```

**Option 2: Port Forwarding (Recommended)**

```bash
kubectl port-forward pod/<pod-name> 8080:8080
curl http://localhost:8080
```

✅ You'll see echo server responding with headers and request info

---

## 🧯 Troubleshooting & Sandbox Tips

> 💡 O'Reilly sandboxes often block external traffic on NodePort

### Common Fixes
- Use port-forwarding instead
- Confirm pod is running
- Check image pull status
- Use internal proxy URL (if provided)

---

## 🚨 Production Guidelines

| Practice         | Sandbox Use      | Production Use                 |
|------------------|------------------|--------------------------------|
| Port-forwarding   | ✔️ Local testing | ❌ Not scalable                |
| NodePort          | ✔️ Basic access  | ❌ Use LoadBalancer or Ingress |
| LoadBalancer      | ❌ Cloud only    | ✔️ External app exposure       |
| YAML deployment   | Optional         | ✔️ Infrastructure as code      |
| RBAC & security   | ❌ Ignored       | ✔️ Must configure roles        |

---

## 💬 Interview Questions

1. What is the role of `kubelet`?
2. Why is etcd critical for the cluster?
3. How does Kubernetes handle failed containers?
4. Difference between ClusterIP, NodePort, and LoadBalancer?

---

## 🎯 Quiz

1. What does “K8s” stand for?
2. Which component assigns Pods to nodes?
3. What command exposes a deployment as a NodePort?
4. What is the purpose of `kubectl port-forward`?

---

## 🖼️ Diagram: How Kubernetes Works

```plaintext
+---------------------+
|  kubectl / CI tools |  <-- User interacts
+----------+----------+
           |
      +----v----+      API Server
      | Control |----------------+
      |  Plane  |                |
      +---------+           +----v-----+
                           | Scheduler |
                           +----+------+
                                |
                    +-----------v----------+
                    |   Worker Node        |
                    | +------------------+ |
                    | | kubelet + Proxy  | |
                    | | Container Runtime| |
                    | +------------------+ |
                    +----------------------+

Pod <-> Service <-> External Access via Port or Proxy
```

---

## 🧑‍💻 Author

- **Name**: Bujji  
- **GitHub**: [Add your GitHub repo link here]  
- **Sandbox**: O'Reilly Virtual Lab  
- **Purpose**: Portfolio-ready, real-time Kubernetes guide  

---

> 🔁 Want to move on to Step 2 (ReplicaSets and scaling)? Say the word, and I’ll prep another hands-on set with `README.md`, challenges, and quiz!

```

---
