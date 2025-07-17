```md
# 🚪 Kubernetes Step 4: Ingress Controllers, Routing & Real-Time Production Practices

## 📚 Overview

Ingress is Kubernetes’ built-in reverse proxy system. It manages **HTTP/HTTPS traffic** by exposing multiple services behind a single IP using path- or hostname-based routing. This step bridges internal services to real-world traffic — simulating production-ready setups.

---

## 🧠 Why Ingress Is Essential in DevOps

| Role of Ingress         | Description                                    | DevOps Benefits                         |
|-------------------------|------------------------------------------------|-----------------------------------------|
| Gateway for traffic     | Handles external HTTP/HTTPS traffic            | Centralized access point                |
| Routing based on rules  | Hostnames or paths mapped to backend services  | Clean URL structure for microservices   |
| SSL termination         | Supports HTTPS using TLS certificates          | Security enforcement                    |
| Traffic shaping         | Rewrite paths, rate-limit, auth enforcement    | Observability, reliability              |

---

## 🕰️ Real-World Scenario

You manage an app with multiple microservices:

- `store.example.com/shop` → Frontend
- `store.example.com/api` → Backend API
- `store.example.com/checkout` → Payment service

Ingress routes traffic based on path rules, balancing load and enforcing HTTPS.

---

## 📦 Hands-On Setup: Sandbox-Compatible

### ✅ 1. Deploy Services

```yaml
# deployment-frontend.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: web
        image: nginx
        ports:
        - containerPort: 80
```

Expose the service:

```bash
kubectl apply -f deployment-frontend.yaml
kubectl expose deployment frontend --port=80
```

Repeat for `backend` and `checkout` using different labels and deployments.

---

### ✅ 2. Install NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml
```

Then verify:

```bash
kubectl get pods -n ingress-nginx
kubectl get svc ingress-nginx-controller -n ingress-nginx
```

---

### ✅ 3. Create Ingress Resource

```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: store-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: store.example.com
    http:
      paths:
      - path: /shop
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-api
            port:
              number: 80
```

Apply:

```bash
kubectl apply -f ingress.yaml
```

Check:

```bash
kubectl get ingress
kubectl describe ingress store-ingress
```

---

### ✅ 4. Simulate External Access in Sandbox

Port-forward the Ingress controller:

```bash
kubectl port-forward svc/ingress-nginx-controller 8080:80 -n ingress-nginx
```

Then in another terminal:

```bash
curl -H "Host: store.example.com" http://localhost:8080/shop
```

✔️ Should return nginx or echo output

---

## 🧯 Production Troubleshooting Checklist

| Problem                  | Command                                | Diagnosis                       | Fix                                 |
|--------------------------|----------------------------------------|----------------------------------|--------------------------------------|
| Ingress not routing      | `kubectl describe ingress`             | Path or service mismatch         | Check service names, ports, paths    |
| No endpoint linked       | `kubectl get endpoints`                | Empty endpoint list              | Verify service selector and Pods     |
| Curl fails with 7 error  | `kubectl port-forward ...`             | Port-forward inactive            | Restart port-forward in new terminal|
| Service returns 404      | `curl` with wrong path                 | Wrong path match                 | Confirm Ingress rule path + rewrite  |
| DNS resolution fails     | `wget` inside Pod                      | Lightweight image lacks DNS      | Use Alpine with `curl` installed     |

---

## 📈 DevOps Principles Applied

| Principle       | Implementation                       |
|-----------------|---------------------------------------|
| Declarative Infra | YAML for Ingress rules and controller |
| Observability     | Logs, metrics, path tracing with Prometheus |
| Automation        | Ingress YAML managed via CI/CD pipeline |
| Portability       | Works across GKE, EKS, AKS, sandbox setups |
| Security          | TLS support, Host rules, RBAC         |
| Resilience        | Routes only to healthy backend Pods   |

---

## 🧠 Interview Questions

### Q1: How does Ingress differ from NodePort?
> Ingress handles HTTP routing via hostname/path; NodePort only exposes one service via static port.

### Q2: What does `nginx.ingress.kubernetes.io/rewrite-target: /` do?
> Ensures URLs match backend service paths, adjusting requests at proxy layer.

### Q3: How would you troubleshoot an Ingress that returns 404?
> Check `kubectl describe ingress`, backend service, endpoints, and Pod health.

### Q4: Why does Ingress stay in `<pending>` state?
> Sandbox lacks cloud provider integration to assign LoadBalancer IP.

---

## 🖼️ Architecture Diagram

```plaintext
User Browser
     |
     v
Ingress Controller (NGINX)
     |
     +-- Host: store.example.com
          |
          +-- Path: /shop     → frontend
          +-- Path: /api      → backend-api
```

---

## 📂 Suggested GitHub Structure

```
kubernetes-ingress/
├── ingress-controller.yaml
├── frontend-deploy.yaml
├── backend-deploy.yaml
├── ingress.yaml
└── README.md
```

---

## 👨‍💻 Author

- **Name**: Bujji
- **Platform**: Kubernetes via O'Reilly Sandbox
- **Focus**: DevOps-aligned real-world Kubernetes workflows
- **GitHub Repo**: [Insert your repo link here]

---


```