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

## ETCD Backup and Restore

### Important Tool

- **etcdctl**: Used for backup operations (`etcdctl snapshot save`)
- **etcdutl**: Used for restore operations (`etcdutl snapshot restore`)

### Prerequisites

Before performing backup/restore operations, identify ETCD configuration:

```bash
# Find ETCD pod configuration
kubectl get pods -n kube-system | grep etcd
kubectl describe pod etcd-master -n kube-system

# Or check static pod manifest
cat /etc/kubernetes/manifests/etcd.yaml
```

Check for the below block:

```yaml
spec:
  containers:
    - command:
        - etcd
        - --advertise-client-urls=https://172.18.0.4:2379 # endpoints
        - --cert-file=/etc/kubernetes/pki/etcd/server.crt # cert
        - --client-cert-auth=true
        - --data-dir=/var/lib/etcd #data-dir
        - --experimental-initial-corrupt-check=true
        - --experimental-watch-progress-notify-interval=5s
        - --initial-advertise-peer-urls=https://172.18.0.4:2380 # initial-advertise-peer-urls
        - --initial-cluster=kind-control-plane=https://172.18.0.4:2380 # initial-cluster
        - --key-file=/etc/kubernetes/pki/etcd/server.key # key
        - --listen-client-urls=https://127.0.0.1:2379,https://172.18.0.4:2379 # endpoints
        - --listen-metrics-urls=http://127.0.0.1:2381
        - --listen-peer-urls=https://172.18.0.4:2380
        - --name=kind-control-plane
        - --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
        - --peer-client-cert-auth=true
        - --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
        - --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
        - --snapshot-count=10000
        - --trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt # cacert
```
### ✅ Which endpoint to use for snapshot?

You should connect to the client endpoint (`--listen-client-urls` or `--advertise-client-urls`) — not the peer or metrics one.

From your config, valid client endpoints are:

```txt
https://127.0.0.1:2379 (loopback)
https://172.18.0.4:2379 (container/host network)
```
Key information needed:

- **Endpoints**:
  - `--endpoints=https://127.0.0.1:2379`
- **Certificates**:
  - `--cacert=/etc/kubernetes/pki/etcd/ca.crt`
  - `--cert=/etc/kubernetes/pki/etcd/server.crt`
  - `--key=/etc/kubernetes/pki/etcd/server.key`
- **Cluster Information**:
  - `--initial-cluster=kind-control-plane=https://172.18.0.4:2380`
  - `--initial-advertise-peer-urls=https://172.18.0.4:2380`
  - `--data-dir=/var/lib/etcd`

### Backup Process

[Documentation - Backup Command ](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#securing-communication)

#### 1. Create Snapshot

```bash
# Create backup snapshot
etcdctl snapshot save /opt/etcd-backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

#### 2. Verify Snapshot

```bash
# With etcdutl (recommended for etcd v3.5+)
etcdutl snapshot status /opt/etcd-backup.db
```

### Restore Process

#### 1. Stop ETCD Service

```bash
# For static pod, move manifest temporarily
sudo mv /etc/kubernetes/manifests/etcd.yaml /etc/kubernetes/manifests/etcd.yaml.bak
```

#### 2. Remove the data-dir

```bash
# Remove old data
rm -rf /var/lib/etcd
# Temp Remove
mv /var/lib/etcd /var/lib/etcd_bak
```

#### 3. Restore from Snapshot

```bash
# Restore using etcdutl (recommended)
etcdutl snapshot restore /opt/etcd-backup.db --data-dir /var/lib/etcd
```

#### 4. Restart ETCD - POD

```bash
# For static pod, restore manifest
sudo mv /etc/kubernetes/manifests/etcd.yaml.bak /etc/kubernetes/manifests/etcd.yaml
```
