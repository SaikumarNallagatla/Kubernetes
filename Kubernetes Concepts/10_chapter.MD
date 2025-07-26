**Kubernetes Resource Requests & Limits**

---

### 🔍 What are Resource Requests and Limits?

In Kubernetes, **Resource Requests and Limits** help manage **CPU and memory usage** of pods to ensure fair allocation of cluster resources.

They are declared in the pod or container YAML specification.

---

### ✅ Why They Are Important for DevOps Engineers

* Prevents a single pod from using all available resources (avoiding cluster crashes).
* Enables **effective resource planning** and cost optimization.
* Required for **auto-scaling (HPA)** to work properly.
* Helps teams **control noisy neighbor problems** in shared environments.

---

### 📑 Anatomy of Resource Requests and Limits

You declare both in each container definition inside a Pod YAML like this:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-demo
spec:
  containers:
    - name: nginx
      image: nginx
      resources:
        requests:
          memory: "64Mi"
          cpu: "250m"
        limits:
          memory: "128Mi"
          cpu: "500m"
```

* **requests**: Minimum amount of resource the container is guaranteed.
* **limits**: Maximum resource the container can use.

If the container tries to use more than the limit, it is throttled (CPU) or killed (memory).

---

### 🔍 Example Explained

```yaml
resources:
  requests:
    cpu: "250m"       # 0.25 CPU core guaranteed
    memory: "64Mi"    # 64 MB memory guaranteed
  limits:
    cpu: "500m"       # max 0.5 CPU core
    memory: "128Mi"   # max 128 MB memory
```

If the pod exceeds **128Mi** memory, it is **OOMKilled (Out of Memory)**.
If it exceeds **500m** CPU, it is **CPU throttled**.

---

### 🚀 Use Case: High Traffic App vs Batch Job

* **Web App (nginx)**: Set higher `requests` to avoid slow responses.
* **Batch Job**: Keep low `requests` and high `limits` to run fast but not always consume CPU.

---

### ❌ What Happens If You Don't Set These?

* Kubernetes may **overcommit** resources.
* Pod may be evicted or OOMKilled during load.
* HPA (Horizontal Pod Autoscaler) won’t function properly.

---

### ❓ Common Doubts Cleared

**1. Is request mandatory?**  ✅ No, but highly recommended. If not set, Kubernetes assumes 0.

**2. Can I set only limit?** ❌ Bad idea. Set both.

**3. What is CPU `250m`?**
`250m` = 0.25 vCPU. `m` stands for millicores.

**4. What if I use more memory than limit?**  ⚠️ Pod will be terminated with `OOMKilled`.

---

### ✍️ Best Practices

* Always define both `requests` and `limits`.
* Monitor usage with tools like **Datadog, Prometheus, or Grafana**.
* Tune values based on real usage and load testing.

---

# 🔄 Horizontal Pod Autoscaler (HPA) in Kubernetes

---

## ✅ What is HPA?

**HPA (Horizontal Pod Autoscaler)** is a Kubernetes controller that automatically adjusts the number of pod replicas in a deployment, replica set, or stateful set **based on observed CPU/memory utilization or custom metrics**.

It helps in scaling your application **horizontally** (by increasing or decreasing the number of pods), ensuring optimal resource usage and performance.

---

## 📈 Why HPA is Important for DevOps Engineers?

As a DevOps engineer, you're responsible for making sure your application is always available and cost-effective:

- You don’t want too **few pods** (which may cause application crashes or slowness).
- You also don’t want too **many pods** (which may waste resources and increase costs).

➡️ **HPA automates this balance** by increasing or decreasing pods based on demand.

---

## ⚙️ How HPA Works Internally

1. **Metrics Server** collects resource usage data (CPU, memory).
2. **HPA controller** continuously watches metrics.
3. If usage exceeds the configured threshold, HPA increases the number of pods.
4. If usage falls below the threshold, HPA scales down the pods.

HPA controller checks every **15 seconds** (by default) for changes.

> 🔍 HPA only works on resources created via Deployment, ReplicaSet, StatefulSet, etc. It won’t work directly on pods.

---

## 📄 Example YAML for HPA (CPU-based)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 60
```

### 🧠 What This YAML Does:
- Links to `myapp-deployment`
- Keeps a **minimum of 2 pods**, and a **maximum of 10 pods**
- When CPU usage goes **above 60%**, it scales up
- When CPU usage goes **below 60%**, it scales down

---

## 🧪 General Example to Understand

Imagine you're running a coffee shop:
- In the morning, you have 2 staff (pods).
- As customers increase, you hire more temporary staff (scale up pods).
- By afternoon, customer traffic reduces, so you let temporary staff go (scale down).

This is exactly how **HPA behaves automatically based on load**.

---

## 🔄 HPA Commands

Enable metrics server:
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Check HPA status:
```bash
kubectl get hpa
kubectl describe hpa myapp-hpa
```

Manually stress the CPU (for testing):
```bash
kubectl run -it --rm load-generator --image=busybox /bin/sh
# Inside the shell:
while true; do wget -q -O- http://<your-service>; done
```

---

## ❓ Common Questions

### Q: Can we autoscale based on memory?
➡️ Yes, just change `name: cpu` to `name: memory` in the YAML.

### Q: Can we scale based on custom metrics?
➡️ Yes, using Prometheus Adapter or custom metrics APIs.

### Q: Can HPA work without a metrics server?
➡️ ❌ No, metrics-server is mandatory.

---

## 📌 Summary

| Feature              | HPA                                          |
|----------------------|-----------------------------------------------|
| Scales               | Number of pods                                |
| Based on             | CPU, Memory, Custom Metrics                   |
| Minimum requirement  | Metrics Server                                |
| Works on             | Deployment, ReplicaSet, StatefulSet           |
| Frequency            | Every 15 seconds                              |

---

> ✅ HPA is an essential tool for managing application load automatically, reducing manual intervention and improving resource efficiency.

Shall we continue with **Vertical Pod Autoscaler (VPA)** next?

# Kubernetes Vertical Pod Autoscaler (VPA)

---

## 🔍 What is Vertical Pod Autoscaler (VPA)?

Vertical Pod Autoscaler (VPA) is a Kubernetes component that **automatically adjusts the CPU and memory (resource requests and limits) of your Pods based on historical usage**. Unlike HPA (which scales out/in), VPA scales **up/down the resources of the pod**.

Think of VPA as a tool that gives your pods the **right amount of fuel (CPU and memory)** they need to run smoothly—**not too much, not too little**.

---

## 🚀 Why Use VPA?

- To **optimize resource usage** (avoid over-provisioning or under-provisioning)
- Improve **application stability** by avoiding OOMKills or CPU throttling
- Useful for **batch jobs, cron jobs, or workloads that can't be horizontally scaled**

---

## ⚙️ How VPA Works Internally

### 🔄 Flow:
1. **VPA Recommender** – Monitors pod resource usage and suggests optimal resources
2. **VPA Updater** – Kills and recreates pods with updated requests/limits
3. **VPA Admission Controller** – Injects updated resources when a new pod is created

> VPA **does not live-update running pods**—it recreates them with new resource settings

---

## 📁 VPA YAML Example

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind:       Deployment
    name:       my-app
  updatePolicy:
    updateMode: "Auto"   # Options: Off, Initial, Auto
  resourcePolicy:
    containerPolicies:
      - containerName: "my-app-container"
        minAllowed:
          cpu: 100m
          memory: 100Mi
        maxAllowed:
          cpu: 1
          memory: 1Gi
```

---

## ✨ Explanation
- `targetRef`: Refers to the deployment you want VPA to manage
- `updateMode: Auto`: VPA will apply recommendations and recreate pods if needed
- `containerPolicies`: Define min/max resource boundaries per container

---

## 📌 Real-World Analogy

Imagine you're organizing lunch boxes for your team:
- Some people eat less, some more
- You observe everyone's eating habits for a week
- Next week, you give each person exactly the amount of food they need (not too much, not too little)

That’s what VPA does for your containers!

---

## 🤔 Common Doubts and Clarifications

### 1. ❓ Can HPA and VPA work together?
- Not directly on the same resource (CPU), but yes with memory or with different policies.
- Best practice: Use HPA for scaling out and VPA for memory tuning.

### 2. ❓ Will VPA restart my pods?
- Yes, it needs to **delete and recreate** pods to apply new settings

### 3. ❓ Is VPA good for all workloads?
- No. For **stateless, high availability** workloads, HPA is usually preferred.
- VPA is better for **batch jobs, cronjobs, or non-critical apps**.

---

## 🧠 Use Cases
- Tuning resource limits for batch-processing apps
- Optimizing memory-heavy workloads (e.g., ML model training)
- Reducing cost in cloud by adjusting over-provisioned containers

---

## 🔁 VPA vs HPA vs Cluster Autoscaler
| Feature               | HPA             | VPA             | Cluster Autoscaler |
|----------------------|------------------|------------------|---------------------|
| Type of Scaling       | Horizontal        | Vertical         | Node-level scaling  |
| Adjusts               | Replica count     | CPU/Memory       | Node pool size      |
| Pod restart required? | No                | Yes              | No                  |
| Best For              | Web apps          | Batch/CronJobs   | Auto node provisioning |

---



