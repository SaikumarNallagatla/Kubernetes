```md
# 🌐 Kubernetes Step 3: Services — Theory, Hands-On & DevOps Practices

## 📚 Table of Contents

1. [Introduction to Services](#introduction-to-services)
2. [Service Types & Real-World Use](#service-types--real-world-use)
3. [How Services Work](#how-services-work)
4. [Sandbox Hands-On Guide](#sandbox-hands-on-guide)
5. [Production Environment Example](#production-environment-example)
6. [Troubleshooting Checklist](#troubleshooting-checklist)
7. [DevOps Principles in Services](#devops-principles-in-services)
8. [Architecture Diagram](#architecture-diagram)
9. [Interview Questions & Answers](#interview-questions--answers)
10. [Appendix: Quiz](#appendix-quiz)

---

## 🧠 Introduction to Services

In Kubernetes, a **Service** is an abstraction that defines a logical set of Pods and a policy by which to access them. Because Pods can die and restart with new IPs, Services allow stable networking and load balancing.

> 📌 Services = networking glue + traffic director

---

## 🧩 Service Types & Real-World Use

| Type            | Description                                 | Real-World Use Case                              |
|------------------|---------------------------------------------|--------------------------------------------------|
| `ClusterIP`       | Internal-only access within cluster          | Microservices backend-to-backend communication   |
| `NodePort`        | Exposes app on static port per node         | Dev/test exposure in labs                        |
| `LoadBalancer`    | Cloud-native public access                  | Real production apps on AWS, GCP, Azure          |
| `ExternalName`    | Maps to external DNS name                   | Legacy app or third-party service integration    |
| Headless Service  | No ClusterIP; direct Pod access             | StatefulSets, DNS-based load balancing           |

---

## 🔍 How Services Work

Kubernetes uses `selectors` to tie a Service to matching Pods. It then creates **endpoints** with Pod IPs and manages load balancing across them using `kube-proxy`.

> 🎯 When traffic hits the Service, kube-proxy redirects it to one of the backend Pods

Example DNS format:
```
<service-name>.<namespace>.svc.cluster.local
```

---

## 💻 Sandbox Hands-On Guide

### 1️⃣ Create a Deployment

```yaml
# deployment-echo.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy
spec:
  replicas: 2
  selector:
    matchLabels:
      app: echo-app
  template:
    metadata:
      labels:
        app: echo-app
    spec:
      containers:
      - name: echo-server
        image: k8s.gcr.io/echoserver:1.4
        ports:
        - containerPort: 8080
```

```bash
kubectl apply -f deployment-echo.yaml
```

---

### 2️⃣ Create ClusterIP Service

```yaml
# service-echo.yaml
apiVersion: v1
kind: Service
metadata:
  name: echo-service
spec:
  selector:
    app: echo-app
  ports:
  - protocol: TCP
    port: 8080
    targetPort: 8080
```

```bash
kubectl apply -f service-echo.yaml
```

---

### 3️⃣ Access Service Without Port Forwarding

```bash
kubectl run test-client --image=alpine -it --restart=Never -- /bin/sh
```

Then inside Pod:

```sh
apk add curl
curl echo-service.default.svc.cluster.local:8080
```

✅ Success: response from echo server inside the cluster.

---

## 🏭 Production Environment Example

### Scenario: Expose Frontend Publicly via LoadBalancer

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-lb
spec:
  selector:
    app: frontend
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```

Used in:
- Cloud-based environments
- Auto-provisioned external IP
- Allows user traffic into cluster safely

### Example Flow:

```plaintext
User Browser --> LoadBalancer IP --> Service --> Frontend Pods
```

---

## 🧯 Troubleshooting Checklist

| Problem                    | Diagnosis Command                          | Fix                                     |
|----------------------------|--------------------------------------------|------------------------------------------|
| Service not reachable      | `kubectl get endpoints <service>`         | Check Pod labels match selector          |
| DNS name fails inside Pod  | `kubectl run` with Alpine, install curl   | Use image with full DNS support          |
| No Pods behind Service     | `kubectl get pods --show-labels`          | Ensure label selector matches correctly  |
| Wrong targetPort mapping   | Inspect YAML and Pod containerPort        | Sync ports: Service port ↔ Pod containerPort |

---

## 🤖 DevOps Principles in Services

| Principle         | Applied In Services                                  |
|-------------------|------------------------------------------------------|
| **Declarative Infra** | YAML definitions define networking setup       |
| **Self-Healing**      | Services auto-update endpoints if Pods change  |
| **Scalability**       | Services route traffic evenly to N Pods         |
| **Portability**       | Works on all cloud providers and on-prem       |
| **Security**          | Network policies, RBAC can restrict exposure   |
| **Observability**     | Integrates with logging/metrics systems        |

---

## 🖼️ Architecture Diagram

```plaintext
+---------------------+
| External User       |
+----------+----------+
           |
     [ LoadBalancer ]        <-- cloud provider, external IP
           |
     [ Service: frontend-lb ] <-- type=LoadBalancer
           |
     [ Pods: frontend-app ]   <-- app containers
```

Internal Microservice-to-Microservice:

```plaintext
Pod A --> Service B --> Pod B
```

---

## 💬 Interview Questions & Answers

### Q1: What does a ClusterIP Service do?
> Provides stable internal networking to a group of Pods.

### Q2: Why might a NodePort not work in a sandbox?
> External firewall blocks exposed ports; sandbox doesn’t support external access.

### Q3: How does Kubernetes route traffic from a Service?
> `kube-proxy` uses iptables or IPVS to route requests to endpoint Pods.

### Q4: What's the difference between `targetPort` and `port`?
> `port` = port exposed by Service; `targetPort` = port inside container.

---

## 📊 Appendix: Quiz

1. What command shows which Pods a Service routes to?
   ```bash
   kubectl get endpoints <service-name>
   ```

2. Which Service type provides internal-only access?
   - [ ] NodePort  
   - [✅] ClusterIP  
   - [ ] LoadBalancer  

3. How do Services track Pods?
   - [✅] Label selector  
   - [ ] Pod name  
   - [ ] IP address  

---

## 🧑‍💻 Author

- **Name**: Bujji  
- **Platform**: Kubernetes via O'Reilly Sandbox  
- **Goal**: Build production-grade, GitHub-visible DevOps learning projects  
- **GitHub**: [Add your repo link here]

---

Want to move on to **Step 4: Ingress Controllers & Routing**, or break this into folders with example YAMLs and push-ready files? You’re constructing a killer Kubernetes blueprint 🚢🔥
```