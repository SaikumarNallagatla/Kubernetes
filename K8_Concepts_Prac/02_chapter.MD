

---

```md
# ⚙️ Kubernetes Step 2: Pods, ReplicaSets & Deployments

## 📚 Overview

This step covers the essential building blocks of Kubernetes workload management:
- Creating Pods manually for core understanding
- Using ReplicaSets for self-healing and scaling
- Managing app lifecycle via Deployments
- Following DevOps principles like declarative config, automation, and version control

---

## 🧠 Why These Concepts Matter

| Concept      | Role in Kubernetes                  | DevOps Alignment              |
|--------------|--------------------------------------|-------------------------------|
| Pod          | Smallest deployable unit             | Container abstraction         |
| ReplicaSet   | Ensures a desired number of Pods     | Resilience & auto-healing     |
| Deployment   | Orchestrates updates, rollback, scale| CI/CD integration, GitOps     |

---

## 🕰️ Historical Context

Before Kubernetes:
- Engineers manually launched Docker containers.
- If one crashed, someone had to restart it.
- Scaling was manual and fragile.

Kubernetes solved this with **Pods** as container groups and **controllers** like **ReplicaSets** and **Deployments** to automate healing, scaling, and rolling updates.

---

## 🧱 Architecture Diagram

```plaintext
+----------------------+
|  kubectl / CI tools  |
+----------+-----------+
           |
      +----v----+
      | API     |
      | Server  |
      +----+----+
           |
+----------v------------+
| Scheduler & Controller|
+----------+------------+
           |
      +----v----+
      | ReplicaSet |
      +----+----+
           |
     +-----v-----+   +-----v-----+
     | Pod 1     |   | Pod 2     |
     | Container |   | Container |
     +-----------+   +-----------+
```

---

## 🔧 Hands-On Practice

### 🔹 Create a Pod (Manually)

```yaml
# pod-nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

```bash
kubectl apply -f pod-nginx.yaml
kubectl get pods
kubectl describe pod nginx-pod
```

✅ Check:
- Pod STATUS: `Running`
- Image pulled successfully
- No errors or crash loops

---

### 🔸 Delete Pod to Understand Ephemeral Nature

```bash
kubectl delete pod nginx-pod
kubectl get pods
```

🧠 Insight: Pods don’t auto-recover unless managed by a controller.

---

### 🔹 Create a ReplicaSet for Auto-Healing

```yaml
# replicaset-nginx.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-demo
  template:
    metadata:
      labels:
        app: nginx-demo
    spec:
      containers:
      - name: nginx
        image: nginx
```

```bash
kubectl apply -f replicaset-nginx.yaml
kubectl get pods
kubectl delete pod <pod-name>
kubectl get pods
```

✅ Check: Pod auto-replaced after deletion

---

### 🔸 Create a Deployment (Preferred DevOps Method)

```yaml
# deployment-nginx.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-web
  template:
    metadata:
      labels:
        app: nginx-web
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
```

```bash
kubectl apply -f deployment-nginx.yaml
kubectl get deployments
kubectl get pods
```

✅ Check: Deployment created ReplicaSet → Pod(s)

---

### 🔄 Rolling Update (DevOps: Zero Downtime)

```bash
kubectl set image deployment/nginx-deploy nginx=nginx:1.20
kubectl rollout status deployment/nginx-deploy
kubectl rollout history deployment/nginx-deploy
kubectl rollout undo deployment/nginx-deploy
```

✅ Version control and rollback complete 🧘

---

## 📋 Checklist for DevOps Principles

| Principle              | Applied In This Step |
|------------------------|----------------------|
| Declarative config     | ✅ YAML files         |
| Automation             | ✅ ReplicaSet, Deployment |
| Version tracking       | ✅ Deployment updates |
| Resilience             | ✅ ReplicaSet auto-healing |
| Portability            | ✅ Container images   |

---

## 💬 Real-Time Interview Questions

1. **What happens when a Pod crashes?**
   - If manually created: it's gone unless redeployed.
   - If managed by ReplicaSet/Deployment: it's auto-recovered.

2. **What’s the difference between ReplicaSet and Deployment?**
   - ReplicaSet maintains Pod count; Deployment manages ReplicaSets with added rollout features.

3. **How do Deployments enable CI/CD?**
   - Can declaratively update container image, track versions, and roll back if needed.

4. **How do you troubleshoot if Pods don’t start?**
   ```bash
   kubectl describe pod <name>
   kubectl logs <name>
   ```

5. **When would you use a Deployment vs a DaemonSet?**
   - Deployment: general app workloads.
   - DaemonSet: one Pod per node (e.g., monitoring agents).

---

## 🎯 Quiz

1. What does a ReplicaSet do in Kubernetes?
2. Can a Deployment exist without a ReplicaSet?
3. What command updates the image of a Deployment?
4. How do you rollback a Deployment to the previous version?
5. How many Pods does a ReplicaSet with replicas: 3 create?

---

## 👨‍💻 Author

- Name: **Bujji**
- GitHub: [Your GitHub Repo Link Here]
- Sandbox: O’Reilly Kubernetes Interactive Lab
- Focus: DevOps-aligned, production-grade learning

---

## 📝 Notes

> ⚠️ Sandbox like O'Reilly may not persist files after timeout — commit work regularly to GitHub  
> ✅ All YAML files follow declarative format recommended in production  
> 💡 Keep each concept in separate folders for versioned documentation and public visibility

---

## 📁 Suggested Folder Structure for GitHub

```
kubernetes-basics/
├── step1-hello-world/
│   ├── pod-nginx.yaml
│   ├── README.md
├── step2-pods-replicasets-deployments/
│   ├── pod-nginx.yaml
│   ├── replicaset-nginx.yaml
│   ├── deployment-nginx.yaml
│   ├── README.md
└── ...
```

---

✅ Let me know if you'd like me to generate this folder’s content for you or move on to **Step 3: Services & Networking**, where you’ll connect everything you've built to the outside world using DNS, LoadBalancers, and cluster communication 🔁🌐
```

---

