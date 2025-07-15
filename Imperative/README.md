# CKA Exam - Essential Imperative Commands

## 📝Quick Note:

Use `--dry-run=client -o yaml > filename.yaml` to quickly generate manifest files and edit them when some flags are not supported.

## 🔧Important Commands:

- `kubectl describe` - Get detailed information about resources
- `kubectl explain` - Show resource documentation and field descriptions
- `kubectl --help` - Get help for any command

  ```bash
  kubectl create svc --help
  kubectl expose deploy --help
  kubectl create service clusterip --help
  ```

**━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━**

## 🚀Pod

```bash
kubectl run my-nginx --image=nginx --port=80 --env="ENV=PROD" --labels="app=nginx-app,env=prod"
```

Common Flags:

- `--restart`= `Always`(Default), `OnFailure`, `Never`
- `--command -- sleep 3600`

**━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━**

## 🚀Deployment

```bash
kubectl create deployment my-dep --image=busybox --replicas=3 --port=80
```

```bash
kubectl set image my-dep busybox=busybox:1.35
```

- `nginx=nginx:1.25` format is `container=image`
- Use `kubectl describe` to get container name

**━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━**

## 🚀Service

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

### **To Create an pod and expose it at the same time**

Use the flag `--expose=true` If true, create a ClusterIP service associated with the pod. Requires `--port`(The port that this container exposes).

```bash
kubectl run pod my-pod --image=nginx --port=80 --expose=tue
```

> **Note**: The above command wil create the `POD` and a `Service` at the same time.

### **Delete and recreate the Pod in single command from manifest file**

```bash
kubectl replace --force -f pod.yaml
```

**━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━**

## 🚀Taints & Tolerations

### Tainting a Node

```bash
kubectl taint nodes node-name key=value:effect
```

Where:

- `effect` can be `NoSchedule`, `PreferNoSchedule`, or `NoExecute`

**Important Flags:**

- `-l, --selector`: Apply taint using label selector
- `--all`: Apply to all nodes in the cluster

### Adding Tolerations to Pods

```yaml
tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "database"
    effect: "NoSchedule"
```

> **Note**: There are only 2 types of operators `Equal` and `Exists`

### **Removing Taints from a Node**

Add a `-` at the end of the same command used for tainting:

```bash
kubectl taint nodes workernode1 app=nginx:NoSchedule-
```

### **Viewing Node Taints**

```bash
kubectl describe nodes | grep -i Taints
```

**━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━**

## 🚀Labels

### **Check node labels**

```bash
kubectl get nodes --show-labels
```

### **Adding a labels**

```bash
kubectl label node <node-name> <key>=<value>
```

### **Removing the label**

```bash
kubectl label node <node-name> <key>-
```

**━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━**

## 🚀Node Affinity

### Types of Node Affinity

1. `requiredDuringSchedulingIgnoredDuringExecution` - Hard requirement (pod won't schedule if no matching node)
2. `preferredDuringSchedulingIgnoredDuringExecution` - Soft preference (scheduler tries but schedules anyway if no match)

### Add Node Affinity to Pod/Deployment

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

## Simple **nodeSelector**

```yaml
spec:
  nodeSelector:
    zone: east
```

**━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━**

## 🚀Resources (Requests & Limits)

The `kubectl set` command provides a quick way to configure resource requests and limits for existing workloads like `Deployments`, `StatefulSets`, `DaemonSets` :

**Basic Usage:**

First, create your resource using any method (`kubectl create` or YAML manifest), then apply resource configurations:

```bash
kubectl set resources deployment nginx-deploy \
  --limits=cpu=1,memory=1Gi \
  --requests=cpu=500m,memory=512Mi
```

> ⚠️ **Important**: This method does not work with standalone Pods, as Pod resource specifications are immutable after creation.

### **Working with Standalone Pods**

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

**━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━**

## 🚀LimitRange

[LimitRange](https://kubernetes.io/docs/concepts/policy/limit-range/) provides default resource limits and requests for containers in a namespace, ensuring consistent resource management across workloads.

**━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━**

## 🚀DaemonSet

DaemonSets cannot be created directly using imperative commands like `kubectl create` or `kubectl run`.

**Workaround: Generate from Deployment**

Since there's no direct imperative command for DaemonSet creation, use this approach:

**Step 1: Generate Deployment YAML**

```bash
kubectl create deployment nginx-daemon --image=nginx --dry-run=client -o yaml > daemonset.yaml
```

**Step 2: Modify the Generated YAML**

Edit the `daemonset.yaml` file and make these changes:

- Change `kind: Deployment` → `kind: DaemonSet`
- Remove the `replicas: 1` line
- Remove the `strategy: {}` section

**Step 3: Apply the DaemonSet**

```bash
kubectl apply -f daemonset.yaml
```

> 💡 **Tip**: DaemonSets automatically run one Pod per node, so replicas and deployment strategies are not applicable.
> **━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━**
