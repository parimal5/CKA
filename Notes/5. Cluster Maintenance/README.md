<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Cluster Maintenance</h3>
</div>

## 🛠️ Node Maintenance & Pod Management

### Safe OS Upgrades on Kubernetes Nodes

- **Ensure Pod Management via Controllers:**
  Always run application pods under controllers like **Deployments** or **ReplicaSets**. These controllers ensure pods are **automatically rescheduled** to healthy nodes during node failure or maintenance.

### 🔁 Node State Management

#### 1. Cordon a Node

- **Purpose:** Marks the node as _unschedulable_.
- **Effect:** No new pods will be scheduled on this node. Existing pods **remain running**.

```bash
kubectl cordon <node-name>
```

#### 2. Draining a Node

- **Purpose:** Prepares a node for maintenance.
- **Effect:**

  - **Evicts** all evictable pods.
  - **ReplicaSet-managed pods** are automatically rescheduled to other nodes.
  - **DaemonSet pods** are ignored unless `--ignore-daemonsets` is specified.
  - **Orphan pods** (not managed by controllers) are deleted **only if** `--force` is used. These are **not rescheduled** and will be **_lost permanently_**.

```bash
kubectl drain <node-name> --ignore-daemonsets
```

#### 3. Uncordon a Node

- **Purpose:** Makes the node _schedulable_ again.
- **Effect:** New pods can be scheduled on the node once more.

```bash
kubectl uncordon <node-name>
```

---

### ⚠️ Important Considerations

- **ReplicaSet-Managed Pods:**
  These are rescheduled automatically on other nodes during draining. This allows **zero-downtime** during upgrades or node shutdowns.

- **Orphan Pods (Static or Standalone):**

  - Not managed by any controller.
  - Deleted during drain only with `--force`.
  - **Not recreated automatically.**
  - Avoid using `--force` unless necessary, and always ensure you have a backup or a way to recreate these pods.

- **DaemonSet Pods:**

  - Not evicted by default during a drain.
  - Add `--ignore-daemonsets` if you want to proceed with draining anyway.

---

## Cluster Upgrade - kubeadm

[**Kubernetes Cluster Upgrade Guide**](./Cluster%20Upgrade/README.md)
