<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Application Lifecycle Management</h3>
</div>

## Rolling Updates and the RollingUpdate Strategy

A Rolling Update is the default update strategy for Deployments. Instead of terminating all old Pods and starting all new ones at once (which can cause downtime), Kubernetes updates Pods incrementally, ensuring some Pods are always available during the process.

### 🛠️ RollingUpdate Strategy Parameters

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1 # Number or percentage of Pods that can be unavailable during the update.
    maxSurge: 1 # Number or percentage of extra Pods (above desired count) that can be created temporarily during the update.
```

### Use Case Example:

You're deploying a new version of your app (v2), replacing the old one (v1):

```bash
kubectl set image deployment my-app my-container=my-app:v2
```

> This triggers a rolling update based on the above strategy.

### 🧯Rollback Support

```bash
kubectl rollout undo deployment my-app
```

### 📋 Checking Rolling Update Status

```bash
kubectl rollout status deployment my-app
```

### 📦 Other Strategies: Recreate Strategy:

Terminates all old Pods first, then creates new ones.

```yaml
strategy:
  type: Recreate
```

## Commands and Arguments

![alt text](image.png)

| Docker Concept | Kubernetes Field | Notes                         |
| -------------- | ---------------- | ----------------------------- |
| `ENTRYPOINT`   | `command`        | Overrides entrypoint in image |
| `CMD`          | `args`           | Passed as args to entrypoint  |
