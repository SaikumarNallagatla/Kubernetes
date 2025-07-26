## Kubernetes Networking Basics

### 🔗 Cluster Networking Overview

Kubernetes networking allows all Pods in a cluster to communicate with each other. It provides IP addresses to each Pod and manages their communication using networking components.

---

### 🌐 Pod-to-Pod Communication

* Every Pod gets a unique IP address
* All Pods can talk to each other without NAT (flat network)
* Pod IPs are ephemeral (they change if Pod restarts)

---

### 🚚 CNI (Container Network Interface)

CNI is a specification and a set of plugins used by Kubernetes to configure networking for containers.

#### ✅ What CNI does:

* Assigns IP address to each Pod
* Manages routing rules
* Ensures connectivity across nodes

#### ❓ Is CNI assigned automatically?

Yes, when you install a Kubernetes cluster, **you must choose a CNI plugin**, or the cluster installation tool (like kubeadm, k3s, or a cloud provider) will install one for you.

#### ✨ Popular CNI Plugins:

* **Flannel** – Simple overlay network
* **Calico** – Advanced, supports NetworkPolicy
* **Weave** – Peer-to-peer overlay
* **Cilium** – eBPF-based, highly scalable

> Example: When you create a cluster with kubeadm, you manually install a CNI like Calico or Flannel.

> Cloud providers like EKS, AKS, and GKE provide default CNI setups automatically.

---

### 📢 Services

Kubernetes Services provide stable networking for Pods:

* **ClusterIP** (default): Internal access only
* **NodePort**: Exposes app on a static port on each Node
* **LoadBalancer**: Provision cloud load balancer
* **ExternalName**: Maps a service to a DNS name

---

### 🧬 Summary

* Pod IPs are given by CNI plugins
* CNI must be installed — either manually or automatically
* Services abstract access to Pods using stable endpoints

---

Next Topic: NetworkPolicy - for controlling who can talk to whom inside the cluster
