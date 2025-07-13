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

## 🚀Pod

```bash
kubectl run my-nginx --image=nginx --port=80 --env="ENV=PROD" --labels="app=nginx-app,env=prod"
```

### Common Flags:

- `--restart`= `Always`(Default), `OnFailure`, `Never`
- `--command -- sleep 3600`

## 🚢Deployment

```bash
kubectl create deployment my-dep --image=busybox --replicas=3 --port=80
```

```bash
kubectl set image my-dep busybox=busybox:1.35
```

- `nginx=nginx:1.25` format is `container=image`
- Use `kubectl describe` to get container name

### Common Flags:

-

## 🌐Service

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

```bash
kubectl expose deployment my-deploy --port=80 --target-port=8080 --name=my-svc --type=NodePort
```

### Common Flags:

- `--type` = `NodePort`, `LoadBalancer`, `ClusterIP` (Default)
- `--labels` = `app=nginx-app`
- `--protocol` = `TCP`, `UDP`, `SCTP`
- `--tcp`=`<port>:<targetPort>`

> **Note**: Can expose Pods, Deployments, RC, etc. using `kubectl expose`

### To Create an pod and expose it at the same time

Use the flag `--expose=true` If true, create a ClusterIP service associated with the pod. Requires `--port`(The port that this container exposes).

```bash
kubectl run pod my-pod --image=nginx --port=80 --expose=tue
```

> **Note**: The above command wil create the `POD` and a `Service` at the same time.

## Delete and recreate the Pod in single command from manifest file

```bash
kubectl replace --force -f pod.yaml
```

## Taints & Tolerations

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

### Removing Taints from a Node

**Add a `-` at the end of the same command used for tainting:**

```bash
kubectl taint nodes workernode1 app=nginx:NoSchedule-
```

### Viewing Node Taints

```bash
kubectl describe nodes | grep -i Taints
```

## Labels

### Quick way to check node labels

```bash
kubectl get nodes --show-labels
```

### Adding a label

```bash
kubectl label node <node-name> <key>=<value>
```

### Removing the label

```bash
kubectl label node <node-name> <key>-
```

## Node Affinity

### Types of Node Affinity

1. **requiredDuringSchedulingIgnoredDuringExecution** - Hard requirement (pod won't schedule if no matching node)
2. **preferredDuringSchedulingIgnoredDuringExecution** - Soft preference (scheduler tries but schedules anyway if no match)

### 2. Add Node Affinity to Pod/Deployment

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

## 3. Alternative: Simple nodeSelector (faster for basic needs)

```yaml
spec:
  nodeSelector:
    zone: east
```

## Common Operators

- `In` - value in list
- `NotIn` - value not in list
- `Exists` - key exists
- `DoesNotExist` - key doesn't exist
