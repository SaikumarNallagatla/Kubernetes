```md
# 🧠 Step 8: Kubernetes Jobs & CronJobs — Automation in DevOps

---

## 📚 Overview

Kubernetes Jobs and CronJobs are controllers designed to handle **automated background tasks**. These are essential for any production-grade app — especially in **e-commerce** where backups, data cleanup, report generation, and cache purging must happen on schedule 🛒⚙️

Jobs → Run once  
CronJobs → Run on schedule (like cron tab)

---

## 🧾 Key Concepts

| Concept   | Purpose                                                       |
|-----------|---------------------------------------------------------------|
| `Job`     | Runs a task to completion (e.g. backup, migration)            |
| `CronJob` | Runs tasks on a recurring schedule (e.g. every night, hourly) |
| `restartPolicy` | Governs retry behavior — typically `OnFailure` or `Never` |
| `volumeMounts` | Used to persist logs or data written by the job          |

---

## 🏪 Real-World Use Case: ShopVerse E-commerce Platform

### 🔹 Background Tasks in Production

| Task                  | Controller | Schedule        | Description                                          |
|-----------------------|------------|------------------|------------------------------------------------------|
| Order cleanup         | CronJob    | Every 3 mins     | Removes stale/abandoned orders from DB               |
| MongoDB backup        | Job        | On-demand        | Exports product catalog for DR and snapshots         |
| Cart cache purging    | CronJob    | Hourly           | Frees up Redis memory by purging expired entries     |
| Sales report generation | CronJob  | Daily at midnight| Generates and uploads daily sales summaries          |
| Image compression     | Job        | After upload     | Optimizes uploaded product images                    |

---

## 🛠️ Hands-On: Job + CronJob Examples

### ✅ 1. One-Time Job — MongoDB Backup Simulation

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: mongo-backup
spec:
  template:
    spec:
      containers:
      - name: backup
        image: busybox
        command: ["sh", "-c", "echo '📦 Backup Complete for ShopVerse DB' | tee /data/backup.log && sleep 5"]
        volumeMounts:
        - name: backup-vol
          mountPath: /data
      restartPolicy: Never
      volumes:
      - name: backup-vol
        hostPath:
          path: /data/shopverse-backup
```

```bash
kubectl apply -f mongo-backup.yaml
kubectl logs mongo-backup-<pod-id>
```

---

### ✅ 2. Scheduled CronJob — Order Cleanup Task

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: order-cleanup
spec:
  schedule: "*/3 * * * *"  # Every 3 mins
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: cleaner
            image: busybox
            command: ["sh", "-c", "echo '🧹 Cleaned orders at: $(date)' >> /data/orders.log && sleep 5"]
            volumeMounts:
            - name: log-vol
              mountPath: /data
          restartPolicy: OnFailure
          volumes:
          - name: log-vol
            hostPath:
              path: /data/order-logs
```

```bash
kubectl apply -f order-cleanup.yaml
kubectl get cronjob
kubectl get jobs
kubectl logs <latest-pod-name>
```

---

## 🖼️ Visual Diagram

```plaintext
     +-----------------------------+
     | CronJob: order-cleanup     |
     +-------------+--------------+
                   ↓ Every 3 mins
         +---------------------+
         | Job: order-cleanup  |
         +----------+----------+
                    ↓
         +---------------------+
         | Pod: cleaner        |
         | Log: /data/orders.log |
         +---------------------+
```

---

## 🧯 Troubleshooting Cheatsheet

| Symptom                           | Likely Cause                                 | Solution                           |
|----------------------------------|----------------------------------------------|------------------------------------|
| `READY 0/1` but `Completed`      | Pod has exited successfully                  | Use `kubectl logs <pod-name>`      |
| Cannot exec into Job Pod         | Pod has completed                            | Use logs, not exec                 |
| No logs appear                   | Container writes only to file, not stdout    | Use `tee` to log & save            |
| Job doesn't start                | YAML error or volume mount issue             | Run `kubectl describe job <name>`  |
| CronJob doesn't run              | Invalid cron format or restartPolicy error   | Validate cron schedule syntax      |

---

## 💬 Interview Questions

### Q1: What’s the difference between a Job and a CronJob?
> Job runs once and completes. CronJob runs on a schedule like traditional cron.

### Q2: Can you exec into a Job Pod after it completes?
> No, you can only use `kubectl logs`. Completed Pods are no longer interactive.

### Q3: How do you persist logs from a Job?
> Use a `volumeMount` to write logs into a mounted volume, or `tee` into both `stdout` and file.

### Q4: What happens if a CronJob pod fails?
> `restartPolicy: OnFailure` ensures retry; or Job may report failure in `.status`

---

## 🛠️ DevOps Principles Applied

| Principle          | Behavior in Kubernetes                        |
|--------------------|-----------------------------------------------|
| Observability      | Jobs output visible via logs + file storage   |
| Automation         | CronJobs run without manual intervention      |
| Resilience         | Restart policies handle failures              |
| Auditing           | Scheduled logs offer trail for governance     |
| CI/CD Integration  | Jobs trigger post-deploy steps or migrations  |

---

## 🧑‍💻 Author

- **Name**: Bujji  
- **Focus**: Stateful infrastructure, automated DevOps workflows  
- **Platform**: Kubernetes O'Reilly Sandbox  
- **Use Case Simulated**: E-commerce background task automation  
- **Next Step**: Step 9 — Kubernetes Probes for Pod Health + Self-Healing

---

Want me to extend this README with a demo on **backup + CronJob alerts**, or prep you for interview questions on CI/CD hooks and cleanup orchestration? You’re engineering scheduled sanity into your systems 🎯📅🚦
```