<h1 align="center"><strong>CKA Exam Notes</strong></h1>
<h2 align="center"><strong>Core Concepts</strong></h2>

---

## 📝 Quick Note

Use `--dry-run=client -o yaml > filename.yaml` to quickly generate manifest files and edit them when some flags are not supported.

## 🔧 Important Commands

- `kubectl describe` - Get detailed information about resources
- `kubectl explain` - Show resource documentation and field descriptions
- `kubectl -h or --help` - Get help for any command

<h2 align="center"><strong>🧩 Pod</strong></h2>

### Creating new Pod

```bash
kubectl run my-nginx --image=nginx --port=80 --env="ENV=PROD" --labels="app=nginx-app,env=prod"
```

#### Common Flags

- `--restart`= `Always`(Default), `OnFailure`, `Never`
- `--command -- sleep 3600`

<h2 align="center"><strong>📦 Deployment</strong></h2>

### Creating new Deployment

```bash
kubectl create deployment my-dep --image=busybox --replicas=3 --port=80
```

### Updating Image

```bash
kubectl set image my-dep busybox=busybox:1.35
```

- `nginx=nginx:1.25` format is `container=image`
- Use `kubectl describe` to get container name

<h2 align="center"><strong>🌐 Service</strong></h2>

### Creating a new Service

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

#### Common Flags

- `--type` = `NodePort`, `LoadBalancer`, `ClusterIP` (Default)
- `--labels` = `app=nginx-app`
- `--protocol` = `TCP`, `UDP`, `SCTP`
- `--tcp`=`<port>:<targetPort>`

### Exposing Existing Resources

```bash
kubectl expose deployment my-deploy --port=80 --target-port=8080 --name=my-svc --type=NodePort
```

> **Note**: Can expose Pods, Deployments, RC, etc. using `kubectl expose`

#### Create Pod and Expose Simultaneously

Use the flag `--expose=true` If true, create a ClusterIP service associated with the pod. Requires `--port`(The port that this container exposes).

```bash
kubectl run pod my-pod --image=nginx --port=80 --expose=true
```

> **Note**: The above command will create the `POD` and a `Service` at the same time.

#### Delete and Recreate Pod from Manifest

```bash
kubectl replace --force -f pod.yaml
```

[Part-2: Scheduling](../Scheduling/README.md)
