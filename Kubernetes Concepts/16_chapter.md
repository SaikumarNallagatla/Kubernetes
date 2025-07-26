# 📌 Kubernetes PodSecurityContext & Security Best Practices

## 🔐 What is `PodSecurityContext`?
A `PodSecurityContext` defines **security settings for all containers** in a Pod. This includes:
- UID (User ID), GID (Group ID)
- FSGroup (for volume access)
- Capabilities
- Privileged mode

> 🧠 It helps enforce **who runs the containers and how securely** they interact with OS-level features.

---

## 🧪 Real Example YAML with `PodSecurityContext`

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000            # All containers will run as this UID
    runAsGroup: 3000           # All containers will use this GID
    fsGroup: 2000              # Volumes will be mounted with this group
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
```

---

## 🔐 Container-level SecurityContext (Overrides Pod level)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: container-sec-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
      securityContext:
        runAsUser: 2000
        allowPrivilegeEscalation: false
        capabilities:
          drop:
            - ALL
```

✅ **runAsUser**: Runs container as user 2000
✅ **capabilities.drop**: Removes root-level Linux capabilities
✅ **allowPrivilegeEscalation**: Prevents container from getting more privileges

---

## ✅ Best Practices (for DevOps)
- ✅ Always drop unused Linux capabilities
- ✅ Avoid running containers as `root` (runAsUser)
- ✅ Use `readOnlyRootFilesystem` where possible
- ✅ Enable AppArmor or Seccomp if available

---

## ❓ Common FAQs

### Q. What happens if you don’t define PodSecurityContext?
> The container may run as `root` by default – risky in shared clusters.

### Q. Is PodSecurityContext inherited by all containers?
> Yes, unless overridden by `securityContext` inside each container.

### Q. Can I enforce cluster-wide security policies?
> Yes, with **Pod Security Admission (PSA)** or **OPA/Gatekeeper**.

---

## 🔍 DevOps Use Cases
- 🔒 Secure critical workloads in production
- 🛡️ Use with RBAC to prevent privilege escalation
- 📦 Combine with Namespaces + NetworkPolicies + PSP (if enabled) for full security model

---

## 🧪 Interview Tip
> **Q:** How would you prevent a container from running as root?
> **A:** Use `runAsUser` in `securityContext` with non-zero UID + drop capabilities.

---

Do you want to continue to **Pod Disruption Budgets (PDB)** next?

## 🎯 Kubernetes Pod Disruption Budgets (PDBs)

### 🔍 What is a Pod Disruption Budget?

A **Pod Disruption Budget (PDB)** is a Kubernetes policy object used to ensure that a **minimum number or percentage of pods remain available** during **voluntary disruptions** (e.g., node drain, cluster upgrade, scaling events).

🔸 It helps maintain **high availability** of applications in production.

> **Voluntary disruptions** include:
>
> * Node upgrades
> * Cluster admin draining a node
> * Evicting pods manually (kubectl delete pod)
>
> **Involuntary disruptions** (like node crashes) are *not* handled by PDBs.

---

### 🧠 Why do we need PDBs?

In production, you can’t afford downtime during planned maintenance. If you are running **critical services with multiple replicas**, you must ensure some pods are always up. PDBs give control over this.

For example, if you have 5 pods and a node is drained (for upgrade), Kubernetes will try to evict pods. Without a PDB, all 5 could be disrupted. With a PDB that allows only 1 pod disruption, K8s will evict only 1 pod at a time, keeping others safe.

---

### 🛠 YAML Example of PDB

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

**Explanation:**

* Ensures **at least 2 pods** are always available.
* Applied to pods with label `app: my-app`

You can also use `maxUnavailable` instead:

```yaml
spec:
  maxUnavailable: 1
```

🔁 Use either `minAvailable` **or** `maxUnavailable`, not both.

---

### ⚙️ How it Works

When a node is drained or a pod is evicted manually, K8s will check the PDB.

* If disruption would violate PDB, K8s blocks the eviction.
* If within limits, K8s allows the eviction.

---

### 📦 Real-World Use Case

#### Scenario:

You have a frontend microservice running 4 replicas (pods) across 3 nodes. During a node upgrade, you don’t want more than 1 pod to be unavailable.

✅ Apply a PDB with:

```yaml
minAvailable: 3
```

➡️ This ensures that **3 out of 4 pods** will remain available always.

---

### 📘 Tips for DevOps Engineers

* Always use PDBs for **stateful apps**, **critical services**, and **frontend workloads**.
* Ensure your **replica count is more than your minAvailable**.
* PDBs help with **zero downtime deployments** when combined with rolling updates.

---

### ❓ Common Doubts

| Question                                                 | Answer                                                      |
| -------------------------------------------------------- | ----------------------------------------------------------- |
| Can PDB prevent involuntary disruptions like node crash? | ❌ No. Only voluntary disruptions.                           |
| Can I use both minAvailable and maxUnavailable?          | ❌ No. Use only one.                                         |
| Is PDB required for all deployments?                     | ⚠️ Not mandatory but **highly recommended** for production. |

---

### ✅ Summary

* PDB ensures high availability during **voluntary disruptions**.
* Defined using either **minAvailable** or **maxUnavailable**.
* Apply on deployments/statefulsets using label selectors.
* Ideal for **HA microservices**, **zero downtime**, and **safe node upgrades**.

---

🚀 Next Topic: **Readiness vs Liveness Probes** – Understanding how Kubernetes knows your app is healthy!
