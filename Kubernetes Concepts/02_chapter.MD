# 🚀 Kubernetes Deep Dive: From Basics to Architecture

This document is designed for absolute clarity and deep understanding of Kubernetes, especially tailored for DevOps engineers and learners preparing for interviews or setting up real-world clusters. It follows a GitHub-friendly format, suitable for Markdown (`README.md`) in VSCode or GitHub repos.

---

## 📦 Docker vs Kubernetes

### Docker:

* A containerization platform that allows you to package an application and its dependencies into a container.
* Provides isolated environments to run apps.

### Kubernetes:

* An orchestration system for managing containerized applications across a cluster of machines.
* Provides features like scaling, rolling updates, self-healing, service discovery.

### 🔍 Difference Table:

| Feature            | Docker                   | Kubernetes                          |
| ------------------ | ------------------------ | ----------------------------------- |
| Purpose            | Containerization         | Container Orchestration             |
| Scope              | Single container runtime | Manages multiple containers & hosts |
| Scaling            | Manual                   | Automatic                           |
| Load Balancing     | No                       | Yes                                 |
| Self-Healing       | No                       | Yes                                 |
| Storage Management | Basic Volumes            | Advanced Persistent Volumes         |

---

## 📜 History of Kubernetes

* **Developed by Google**: Based on Google’s internal system **Borg**.
* **Open-sourced in 2014** and donated to **CNCF** (Cloud Native Computing Foundation).
* Written in **Go (Golang)**.
* Became the industry standard for container orchestration.

> Before Kubernetes, companies used tools like **Docker Compose**, **manual scripting**, and **homegrown Bash solutions** for orchestration.

---

## 🎯 Why Kubernetes? (Before & After)

### Before Kubernetes:

* Manual container starts with `docker run`.
* Difficulty in handling failures or rescheduling containers.
* No service discovery, no built-in scaling.
* No standardized health checks or rollback mechanisms.

### With Kubernetes:

* **Declarative Management**: Use YAML manifests to define desired state.
* **Self-Healing**: Automatically restarts failed containers.
* **Auto-scaling**: Scale apps based on usage.
* **Rolling Updates**: Zero downtime deployments.
* **Service Discovery & Load Balancing** built-in.

---

# 🏗️ Kubernetes Architecture Deep Dive

A Kubernetes cluster has **two parts**:

1. **Control Plane** (Master Node)
2. **Worker Nodes**

### 📊 Visual Architecture Diagram

![Kubernetes Architecture Diagram](./architecture.png)

---

## 🧠 Control Plane Components (Master)

### 1. 📌 `kube-apiserver`

* **Role**: Central entry point to Kubernetes. It handles all REST requests.
* **Think of it like**: The receptionist or API gateway for your cluster.

#### 🔧 Responsibilities:

* Accepts requests from `kubectl`, CI/CD tools like Jenkins or Argo.
* Exposes Kubernetes REST API endpoints.
* Authenticates and validates requests.
* Communicates with `etcd`, `scheduler`, and `controller-manager`.

#### ✅ DevOps Example:

```bash
kubectl apply -f deployment.yaml
```

* This request first hits `kube-apiserver`, which authenticates it, stores the desired state in `etcd`, and then triggers the scheduler.

#### 🤖 Analogy:

* Like a **restaurant waiter** taking orders and sending them to the kitchen (other components).

---

### 2. 📌 `etcd`

* **Role**: A distributed, consistent key-value store for all cluster data.
* **Think of it like**: A **database** that stores the desired and current state of the cluster.

#### 🔧 Responsibilities:

* Stores all configuration data.
* Keeps records of deployments, pods, nodes, secrets, etc.
* Highly available and consistent.

#### ✅ DevOps Example:

* When you deploy an app, `kube-apiserver` saves that state in `etcd`.
* If your cluster crashes, `etcd` is used to restore the entire system.

#### 🤖 Analogy:

* Like a **ledger or database** that remembers every state your system needs.

---

### 3. 📌 `kube-scheduler`

* **Role**: Assigns Pods to suitable Nodes.
* **Think of it like**: A **matchmaker** deciding which worker node should run which pod.

#### 🔧 Responsibilities:

* Watches for unscheduled Pods.
* Selects appropriate Nodes based on:

  * CPU, memory availability
  * Taints & tolerations
  * Node selectors & affinity rules

#### ✅ DevOps Example:

* You define a Pod with high CPU need → scheduler finds a node with enough CPU and assigns it there.

#### 🤖 Analogy:

* Like a **hotel receptionist** assigning guests (pods) to suitable rooms (nodes).

---

### 4. 📌 `kube-controller-manager`

* **Role**: Runs controllers to reconcile the actual state with the desired state.
* **Think of it like**: A **system manager** monitoring everything.

#### 🔧 Responsibilities:

* Runs different controllers:

  * Node Controller
  * Replication Controller
  * Deployment Controller
* Keeps system in desired state.

#### ✅ DevOps Example:

* You set `replicas: 3` in a Deployment. If one Pod fails, controller ensures another is created.

#### 🤖 Analogy:

* Like a **supervisor** constantly checking things are working as expected and correcting them if not.

---

### 5. 📌 `cloud-controller-manager`

* **Role**: Manages interaction with cloud providers.
* **Think of it like**: A **bridge between Kubernetes and the cloud infrastructure**.

#### 🔧 Responsibilities:

* Creates load balancers in AWS/GCP.
* Manages cloud-specific routes, volumes, and instances.

#### ✅ DevOps Example:

* When you expose a service with `type: LoadBalancer`, this component provisions an external IP.

#### 🤖 Analogy:

* Like a **cloud technician** who sets up networking and resources as needed.

---

## ⚙️ Worker Node Components

### 1. 📌 `kubelet`

* **Role**: Node-level agent that manages container lifecycle.
* **Think of it like**: A **local site manager** on each worker node.

#### 🔧 Responsibilities:

* Watches for Pod specs from API server.
* Creates containers using Container Runtime (e.g., containerd).
* Reports health & status back to control plane.

#### ✅ DevOps Example:

* `kubelet` starts Nginx container when API server assigns a Pod to that node.

#### 🤖 Analogy:

* Like a **factory floor manager** making sure instructions from HQ are followed.

---

### 2. 📌 `kube-proxy`

* **Role**: Manages networking rules for Pods and Services.
* **Think of it like**: A **network router** that knows where to send requests.

#### 🔧 Responsibilities:

* Forwards requests to the right Pod.
* Implements service load balancing via iptables or IPVS.

#### ✅ DevOps Example:

* When a frontend app hits `http://my-service`, `kube-proxy` ensures it reaches one of the backend Pods.

#### 🤖 Analogy:

* Like a **traffic cop** directing cars (requests) to the correct buildings (pods).

---

### 3. 📌 Container Runtime (e.g., containerd)

* **Role**: Executes containers on a node.
* **Think of it like**: The **engine that runs containers**.

#### 🔧 Responsibilities:

* Pulls images (e.g., from Docker Hub).
* Starts/stops containers.
* Manages container filesystem and networking.

#### ✅ DevOps Example:

* `kubelet` asks containerd to start a `nginx` container based on your YAML definition.

#### 🤖 Analogy:

* Like a **chef** cooking based on recipes (images), inside a separate kitchen (container).

---

## 🔄 Overall Workflow Example

1. `kubectl apply -f deployment.yaml`
2. `kube-apiserver` validates and stores in `etcd`
3. `kube-scheduler` assigns Pod to a Node
4. `kubelet` on that Node reads the spec
5. `kubelet` tells containerd to pull the image and run it
6. `kube-proxy` handles incoming requests

---

Would you like to continue with YAML files, Helm, or Pod lifecycle next?
