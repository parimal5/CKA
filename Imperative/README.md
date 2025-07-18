# CKA Exam - Essential Imperative Commands

## 📝Quick Note:

Use `--dry-run=client -o yaml > filename.yaml` to quickly generate manifest files and edit them when some flags are not supported.

## 🔧Important Commands:

- `kubectl describe` - Get detailed information about resources
- `kubectl explain` - Show resource documentation and field descriptions
- `kubectl -h or --help` - Get help for any command

  ```bash
  kubectl create svc -h
  kubectl expose deploy -h
  kubectl create service clusterip -h
  ```

---

<h2 align="center"><strong> 🧩 Pod</strong></h2>

```bash
kubectl run my-nginx --image=nginx --port=80 --env="ENV=PROD" --labels="app=nginx-app,env=prod"
```

Common Flags:

- `--restart`= `Always`(Default), `OnFailure`, `Never`
- `--command -- sleep 3600`

---

<h2 align="center"><strong> 📦 Deployment</strong></h2>

```bash
kubectl create deployment my-dep --image=busybox --replicas=3 --port=80
```

```bash
kubectl set image my-dep busybox=busybox:1.35
```

- `nginx=nginx:1.25` format is `container=image`
- Use `kubectl describe` to get container name

---

<h2 align="center"><strong>🌐 Service</strong></h2>

```bash
kubectl create service clusterip my-svc --clusterip="10.25.0.2" --tcp=80:8080
```

```bash
kubectl create service loadbalancer my-svc --tcp=80:8080
```

```bash
kubectl create service nodeport my-svc --node-port=30080 --tcp=80:8080
```

> **Note**: The above commands create services when the target resources (POD, Deployments, RC etc.) are not created.

Common Flags:

- `--type` = `NodePort`, `LoadBalancer`, `ClusterIP` (Default)
- `--labels` = `app=nginx-app`
- `--protocol` = `TCP`, `UDP`, `SCTP`
- `--tcp`=`<port>:<targetPort>`

```bash
kubectl expose deployment my-deploy --port=80 --target-port=8080 --name=my-svc --type=NodePort
```

> **Note**: Can expose Pods, Deployments, RC, etc. using `kubectl expose`

#### **To Create an pod and expose it at the same time**

Use the flag `--expose=true` If true, create a ClusterIP service associated with the pod. Requires `--port`(The port that this container exposes).

```bash
kubectl run pod my-pod --image=nginx --port=80 --expose=tue
```

> **Note**: The above command wil create the `POD` and a `Service` at the same time.

#### **Delete and recreate the Pod in single command from manifest file**

```bash
kubectl replace --force -f pod.yaml
```

---

<h2 align="center"><strong>⚠️ Taints & 🛡️ Tolerations</strong></h2>

#### Tainting a Node

```bash
kubectl taint nodes node-name key=value:effect
```

Where:

- `effect` can be `NoSchedule`, `PreferNoSchedule`, or `NoExecute`

**Important Flags:**

- `-l, --selector`: Apply taint using label selector
- `--all`: Apply to all nodes in the cluster

#### Adding Tolerations to Pods

```yaml
tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "database"
    effect: "NoSchedule"
```

> **Note**: There are only 2 types of operators `Equal` and `Exists`

#### **Removing Taints from a Node**

Add a `-` at the end of the same command used for tainting:

```bash
kubectl taint nodes workernode1 app=nginx:NoSchedule-
```

#### **Viewing Node Taints**

```bash
kubectl describe nodes | grep -i Taints
```

---

<h2 align="center"><strong> 🏷️Labels</strong></h2>

#### **Check node labels**

```bash
kubectl get nodes --show-labels
```

#### **Adding a labels**

```bash
kubectl label node <node-name> <key>=<value>
```

#### **Removing the label**

```bash
kubectl label node <node-name> <key>-
```

---

<h2 align="center"><strong>📍Node Affinity</strong></h2>

#### Types of Node Affinity

1. `requiredDuringSchedulingIgnoredDuringExecution` - Hard requirement (pod won't schedule if no matching node)
2. `preferredDuringSchedulingIgnoredDuringExecution` - Soft preference (scheduler tries but schedules anyway if no match)

#### Add Node Affinity to Pod/Deployment

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: zone
                operator: In
                values:
                  - east
```

Common Operators

- `In` - value in list
- `NotIn` - value not in list
- `Exists` - key exists
- `DoesNotExist` - key doesn't exist

---

<h2 align="center"><strong>📍Simple **nodeSelector**</strong></h2>

```yaml
spec:
  nodeSelector:
    zone: east
```

---

<h2 align="center"><strong>⚖️Resources (Requests & Limits)</strong></h2>

The `kubectl set` command provides a quick way to configure resource requests and limits for existing workloads like `Deployments`, `StatefulSets`, `DaemonSets` :

**Basic Usage:**

First, create your resource using any method (`kubectl create` or YAML manifest), then apply resource configurations:

```bash
kubectl set resources deployment nginx-deploy \
  --limits=cpu=1,memory=1Gi \
  --requests=cpu=500m,memory=512Mi
```

⚠️**Important**: This method does not work with standalone Pods, as Pod resource specifications are immutable after creation.

<details>
<summary><strong>📖 Expand: Working with Standalone Pods⬇️</strong></summary>

Since Pods are immutable regarding their `spec.containers.resources`, you'll need to use the force replace method:

**Step 1: Edit the Pod**

```bash
kubectl edit pod <pod-name>
```

**Step 2: Handle the Expected Error**

When you modify resources, you'll encounter this error on save:

```bash
Error from server (Invalid): Pod "mypod" is invalid:
spec.containers[0].resources: Forbidden: may not be changed directly
```

**_The editor will save your changes to a temporary file:_**

```bash
Edit cancelled, your changes have been saved to:
/tmp/kubectl-edit-XXXX.yaml
```

**Step 3: Force Replace**

Apply your changes using the temporary file:

```bash
kubectl replace --force -f /tmp/kubectl-edit-XXXX.yaml
```

> 💡 **Tip**: The `--force` flag deletes the existing Pod and recreates it with the new configuration, which will cause a brief downtime.

</details>

---

<h2 align="center"><strong>📏LimitRange</strong></h2>

LimitRange provides default resource limits and requests for containers in a namespace, ensuring consistent resource management across workloads.

✅ Purpose:

- Set default CPU/memory requests and limits if not defined by user
- Enforce min/max bounds for container resources

🧱 Resource Types:

- Container (most common)
- Pod
- PersistentVolumeClaim (for storage limits)

---

<h2 align="center"><strong>🔧DaemonSet</strong></h2>

### DaemonSets cannot be created directly using imperative commands like `kubectl create` or `kubectl run`.

<details>
<summary><strong>📖 🔧 DaemonSet Creation Process – Click to Expand⬇️</strong></summary>

Since there's no direct imperative command for DaemonSet creation, use this approach:

#### Step 1: Generate Deployment YAML

```bash
kubectl create deployment nginx-daemon --image=nginx --dry-run=client -o yaml > daemonset.yaml
```

#### Step 2: Modify the Generated YAML

Edit the `daemonset.yaml` file and make these changes:

- Change `kind: Deployment` → `kind: DaemonSet`
- Remove the `replicas: 1` line
- Remove the `strategy: {}` section

#### Step 3: Apply the DaemonSet

```bash
kubectl apply -f daemonset.yaml
```

> 💡 **Tip**: DaemonSets automatically run one Pod per node, so replicas and deployment strategies are not applicable.

</details>

---

<h2 align="center"><strong>⚙️Static POD</strong></h2>

Static PODs are managed directly by the kubelet on each node, bypassing the Kubernetes `API server, scheduler, etcd, and controllers`. This makes them useful for running critical system components that need to be available even when the control plane is unavailable.

**Key Characteristics:** Node-specific , Kubelet-managed , Pod-only (no Deployments, DaemonSets, ReplicaSets, etc.) , Auto-restart

#### 📁 Default Path

- `/etc/kubernetes/manifests`

#### 📁 Custom Path

The static POD path can be customized by modifying the kubelet configuration file: `/var/lib/kubelet/config.yaml`

**Configuration field in config.yaml:**

```yaml
staticPodPath: /etc/kubernetes/manifests  # Default path
# or
staticPodPath: /custom/path/to/manifests  # Custom path
```

📌 Behavior

- Add YAML to path(staticPodPath) → pod auto-created
- Deleting pod via kubectl = ineffective (auto-recreated)
- Delete the YAML file to permanently stop the pod

<details>
<summary><strong>📖 Expand: Identifying Static PODs vs Regular PODs ⬇️</summary></strong>

#### 1. Naming Convention

Static PODs have the node name appended to their name:

```bash
kubectl get pods -A

NAMESPACE     NAME                           READY   STATUS    RESTARTS   AGE
kube-system   etcd-master-worker-node01      1/1     Running   0          10m      # Static POD
kube-system   kube-apiserver-control-plane   1/1     Running   0          10m      # Static POD
kube-system   coredns-558bd4d5db-xyz123      1/1     Running   0          10m      # Regular POD
kube-system   kube-proxy-abc456              1/1     Running   0          10m      # Regular POD
```

> **Pattern**: `<pod-name>-<node-name>` indicates a static POD

#### 2. Check POD Details with kubectl describe

```bash
kubectl describe pod <pod-name> -n <namespace>
```

**For Static PODs:**

```yaml
Name: etcd-master-node
Controlled By: Node/master-node # Key indicator
```

**For Regular PODs:**

```yaml
Name: nginx-deployment-789def-12345
Controlled By: ReplicaSet/nginx-deployment-789def # Regular POD controlled by ReplicaSet
```

#### 3. Check ownerReferences

```bash
kubectl get pod <pod-name> -n <namespace> -o yaml | grep -A 5 ownerReferences
```

**Static POD output:**

```yaml
ownerReferences:
  - apiVersion: v1
    controller: true
    kind: Node # Owned by Node
    name: master-node
```

**Regular POD output:**

```yaml
ownerReferences:
  - apiVersion: apps/v1
    controller: true
    kind: ReplicaSet # Owned by ReplicaSet/Deployment
    name: nginx-deployment-789def
```

</details>

---

<h2 align="center"><strong>🎯PriorityClass</strong></h2>

- Defines a priority value for Pods (higher = more important)
- Helps Kubernetes schedule and evict pods during resource pressure

#### Creating the `PriorityClass`

```bash
kubectl create pc high-prio --value=1000 --global-default=false --preemption-policy='PreemptLowerPriority'
```

#### preemptionPolicy Values

- `PreemptLowerPriority` → Can evict lower-priority pods (**Default**) ✅
- `Never` → Waits for resources, won’t evict anyone ❌

#### Assingnt the PriorityClass

```yaml
spec:
  priorityClassName: high-priority
  preemptionPolicy: Never # ✅ Optional (This overide the default values)
```

> **NOTE**: `priorityClassName` and `preemptionPolicy` are always applied at the `Pod` level, even when using `Deployments`, `DaemonSets`, or other controllers — set them inside `spec.template.spec`.

> _IMPORTANT_: When you create a pod, Kubernetes automatically adds a computed priority field (e.g., `priority: 0`) based on the `priorityClassName`. If you're using `kubectl edit` or applying a modified pod YAML, make sure to keep only `priorityClassName` — do not include the priority field, or it will cause a `Forbidden error`.

---

<h2 align="center"><strong>🕒 Multiple-Schedulers</strong></h2>

Kubernetes allows you to run multiple schedulers simultaneously within a cluster. This enables you to use specialized scheduling logic for different workloads while maintaining the default scheduler for standard operations.

Schedular can be run as POD, Deployment or ReplicaSet etc.

#### 1. List Available Schedulers

```bash
kubectl get pods -n kube-system
kubectl get pods -n kube-system -l component=kube-scheduler
```

#### 2. Using Custom Scheduler in Pod

```yaml
spec:
  schedulerName: my-custom-scheduler
```

#### 3. Validation Commands

```bash
# Check which scheduler was used
kubectl describe pod <pod-name> | grep "Scheduled"

# Check all events
kubectl get events -o wide
```

---

<h2 align="center"><strong>Admission Controllers</strong></h2>

### Overview

Admission Controllers allow you to modify the behavior of resources in your Kubernetes cluster. They intercept requests to the API server before objects are persisted. They are gatekeepers that run after authentication/authorization but before objects are stored in etcd.

#### Built-in Examples

- **NamespaceAutoProvision** - Automatically creates namespaces if they don't exist
- **DefaultStorageClass** - Automatically assigns default storage class to PVCs
- **NamespaceExists** - Validates that the specified namespace exists (throws error if not)

### Configuration

Admission controllers are managed by the `kube-apiserver` and some are enabled by default.

#### Viewing Current Configuration

Check the current admission controller configuration:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Look for these fields:

```yaml
- --enable-admission-plugins=plugin1,plugin2
- --disable-admission-plugins=plugin1,plugin2
```

### Listing Available Plugins

To see all available admission controllers:

```bash
kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep "enable-admission-plugins"
```

### Key Points

- Admission controllers run as part of the kube-apiserver
- They can be enabled or disabled by editing the kube-apiserver static pod manifest
- Configuration requires restarting the kube-apiserver
- Some controllers are enabled by default in most clusters

> **NOTE**: Editing admission controller settings requires kube-apiserver restart (automatic if static pod on control plane).

<h2 align="center"><strong>Miscellaneous</strong></h2>

### kubectl `--command` Flag Positioning

**Key Rule**: `--command` must be the **LAST** kubectl option before `--`

✅ **Correct**:

```bash
kubectl run pod --image=busybox --dry-run=client -o yaml --command -- sleep 1000
# kubectl options first, then --command, then container command
# OR
kubectl run pod --image=busybox --dry-run=client -o yaml --command -- sleep 1000 > pod.yaml
```

❌ **Wrong**:

```bash
kubectl run pod --image=busybox --command -- sleep 1000 --dry-run=client -o yaml
# Everything after -- becomes container command: "sleep 1000 --dry-run=client -o yaml"
```

> **Remember**: Everything after `--` is treated as the container command, not kubectl options.
