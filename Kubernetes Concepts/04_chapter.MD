## Kubernetes Networking - Services

### 🌐 What is a Service in Kubernetes?

A **Service** in Kubernetes is a logical abstraction that defines a set of Pods and a policy by which to access them. Since Pods can die and be recreated (with a different IP), Services provide a **stable endpoint** to access applications running in the cluster.

---

🧠 **Think of it like:**
A **Service** is like a **hotel receptionist**. Guests (users) come in and ask for Room 101, but behind the scenes, Room 101 may have changed floors or rooms. The receptionist (Service) knows where to route the request every time.

---

🔧 **Types of Services:**

* **ClusterIP (default):** Accessible only within the cluster.
* **NodePort:** Exposes the service on a static port on each node.
* **LoadBalancer:** Provisions a load balancer from the cloud provider.
* **ExternalName:** Maps service to an external DNS name.

---

✅ **DevOps Use Case:**
You deployed 3 Pods running your backend service. They can be replaced anytime, so their IPs change. You create a **Service** to expose those Pods at a fixed DNS name (`my-backend-svc`).

```bash
kubectl apply -f service.yaml
```

Now your frontend can always call `http://my-backend-svc:8080` without worrying about Pod IP changes.

---

🤖 **Simple Analogy:**
Imagine you run a restaurant. You don’t tell customers the name of the current chef in the kitchen (Pod IP); instead, they order food via the **waiter** (Service), who always knows which chef is on duty.

---

📄 **Example service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-backend-svc
spec:
  selector:
    app: my-backend
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
  type: ClusterIP
```

This exposes the Pods with label `app: my-backend` on port 8080 internally.

---

🛠️ **Production Use Case:**

* Your Pods run a Flask API server (port 80).
* You expose them via a Service on port 8080.
* Any internal app can now access the Flask service via `my-backend-svc:8080`.

---

## 🌐 What is Ingress in Kubernetes?

### 🚦 Ingress - The Smart Router for HTTP Traffic

An **Ingress** is a Kubernetes object that manages **external access to services**, typically HTTP/HTTPS. It acts like a smart router at the edge of your cluster, directing incoming traffic to the right Service based on the request’s hostname or path.

---

🧠 **Think of it like:**
An **Ingress** is like a receptionist in a large office building. Depending on the company name (host) or purpose (URL path), the receptionist sends you to the right floor (Service).

---

🔧 **Key Features:**

* Routes traffic based on domain name (`host`) and/or URL path (`/api`, `/login`)
* Supports TLS (HTTPS)
* Can use custom Ingress Controllers like NGINX, Traefik, Istio
* Reduces the need for multiple LoadBalancers or NodePorts

---

✅ **DevOps Example:**
Let’s say you have two services:

* `frontend-svc` (UI)
* `backend-svc` (API)

You want:

* `/` → goes to frontend
* `/api` → goes to backend

You create an **Ingress** object with rules:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: myapp.company.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-svc
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend-svc
            port:
              number: 80
```

This exposes two services under the same domain.

---

🤖 **Simple Analogy:**
Ingress is like a receptionist who guides people depending on what they ask: “I’m here for support” → backend; “I want to browse products” → frontend.

---

🛠️ **Real-world Scenario:**
In production, you have:

* A web UI
* An API backend
* A monitoring dashboard

You create one Ingress, attach your domain, and route all services through it. You use **cert-manager** with Let’s Encrypt to enable TLS (HTTPS) without manual intervention.

---

🔒 **Bonus: Securing with HTTPS**
Ingress works with TLS to enable HTTPS:

```yaml
  tls:
  - hosts:
    - myapp.company.com
    secretName: tls-secret
```

You must create `tls-secret` with your cert and key.

---

Would you like to continue with **ConfigMap & Secret** next?
