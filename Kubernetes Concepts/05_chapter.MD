# 📦 Kubernetes ConfigMap, Secret, and Volume Deep Dive

This repository explains in-depth the use of **ConfigMap**, **Secret**, and **Volumes** in Kubernetes. It is targeted at both beginners and DevOps Engineers who want to master Kubernetes configuration management and pod data injection.

---

## 🔹 What is ConfigMap?

* ConfigMap is used to store **non-sensitive configuration data** in key-value pairs.
* Useful for environment-specific config, app settings, JSON, YAML, and `.properties` files.

### ✅ Why Use ConfigMap?

* Decouples config from code
* Easily updateable without rebuilding the image
* Supports reusability across multiple pods

---

## 🔹 What is Secret?

* Secret is used to store **sensitive information** like passwords, tokens, TLS certs.
* Values are base64 encoded (not encrypted by default).

### ✅ Why Use Secrets?

* Protect sensitive data from being exposed in plaintext
* Follows RBAC to restrict access

---

## 🔹 ConfigMap vs Secret

| Feature         | ConfigMap         | Secret            |
| --------------- | ----------------- | ----------------- |
| Data Type       | Non-sensitive     | Sensitive         |
| Encoding        | Plain Text        | Base64 encoded    |
| Use Case        | App config, modes | Passwords, tokens |
| Volume Support  | ✅ Yes             | ✅ Yes             |
| Env Var Support | ✅ Yes             | ✅ Yes             |

---

## 🔹 Usage Methods

### 1. As Environment Variables

```yaml
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  APP_MODE: production
  MAX_USERS: "500"
```

```yaml
# Deployment
envFrom:
  - configMapRef:
      name: app-config
```

```yaml
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ=
```

```yaml
# Deployment
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_PASSWORD
```

### 2. As Volume Mounts

```yaml
# Mount ConfigMap
volumes:
  - name: config-volume
    configMap:
      name: app-config

volumeMounts:
  - name: config-volume
    mountPath: /etc/config
```

```yaml
# Mount Secret
volumes:
  - name: secret-volume
    secret:
      secretName: app-secret

volumeMounts:
  - name: secret-volume
    mountPath: /etc/secret
```

---

## 🧱 Volumes Deep Dive

### 🔹 What is a Volume?

* A Volume in Kubernetes is a directory accessible to containers in a pod.
* Used to **persist data**, share files between containers, and inject config.

### 🔹 Types of Volumes:

* `emptyDir`: Temporary, exists as long as the pod lives.
* `hostPath`: Mounts a file/directory from the host node.
* `configMap` / `secret`: Mounts data as files.
* `persistentVolumeClaim`: Used for persistent data.

---

## 🔹 Persistent Volume (PV) & Persistent Volume Claim (PVC)

### 🔹 What is a Persistent Volume (PV)?

* PV is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using StorageClasses.
* It is like a storage resource in the cluster.

### 🔹 What is a Persistent Volume Claim (PVC)?

* PVC is a request for storage by a user.
* It specifies size, access modes, and storage class.
* PVC binds to an available PV.

### ✅ Why use PVC/PV?

* To **persist** data beyond pod lifecycle (e.g., database storage).

### 🧾 PVC YAML Example:

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

### Mounting PVC in Pod:

```yaml
volumes:
  - name: pvc-storage
    persistentVolumeClaim:
      claimName: demo-pvc

volumeMounts:
  - mountPath: /data
    name: pvc-storage
```

---

## 🔹 ConfigMap and Secret as File vs ENV

| Feature             | ENV VAR                 | File Mount                      |
| ------------------- | ----------------------- | ------------------------------- |
| Usage               | Small text configs      | Certificates, keys, large files |
| Accessed by app     | `process.env.KEY`       | `/etc/config/key`               |
| Dynamic reload      | ❌ No (static)           | ✅ Yes (some controllers reload) |
| File format support | ❌ Only plain key-values | ✅ Any file format (json, .crt)  |
| Limitation          | 1MB size total          | 1MB per key                     |

---

## 🔐 YAML Examples: ConfigMap + Secret + Volume Mounts

### ConfigMap:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: demo-config
data:
  APP_PORT: "3000"
  APP_ENV: "dev"
```

### Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: demo-secret
type: Opaque
data:
  API_KEY: YXBpX3Rva2Vu
```

### Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-secret-pod
spec:
  containers:
    - name: nginx
      image: nginx
      envFrom:
        - configMapRef:
            name: demo-config
      env:
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: demo-secret
              key: API_KEY
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
        - name: secret-vol
          mountPath: /etc/secrets
        - name: pvc-storage
          mountPath: /data
  volumes:
    - name: config-vol
      configMap:
        name: demo-config
    - name: secret-vol
      secret:
        secretName: demo-secret
    - name: pvc-storage
      persistentVolumeClaim:
        claimName: demo-pvc
```

---

## 📌 DevOps Notes:

* Use ENV VARs for small, app-level configs
* Use Volume Mounts for files, TLS certs, and reloadable configs
* Use PVCs when you want to persist app data (databases, uploads)
* Secrets should not be committed to GitHub (use `.gitignore`, external vaults)
* Use `kubectl describe pod <pod-name>` to debug config mounts

---

## 💬 Common Errors:

| Issue                      | Fix                                                             |
| -------------------------- | --------------------------------------------------------------- |
| ConfigMap/Secret not found | Check `name:` and namespace                                     |
| CrashLoopBackOff           | Check logs with `kubectl logs` – may be bad ENV or volume mount |
| Wrong mount path           | Validate mountPath is correct in container                      |
| PVC not bound              | Check `kubectl get pvc` and ensure a PV is available            |

---

> Maintained by: [Charan](https://github.com/charan-devops)
> Language: English
> Format: Production-Ready with Best Practices ✅
