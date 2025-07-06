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
  kubectl create service clusterip
  ```

## 🚀Pod

```bash
kubectl run my-nginx --image=nginx --port=80 --env="ENV=PROD" --labels="app=nginx-app,env=prod"
```

### Common Flags:

- `--restart`=`Never`
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
- `--protocol` = `TCP|UDP|SCTP`
- `--tcp`=`<port>:<targetPort>`

### Note:

- Can expose Pods, Deployments, RC, etc. using `kubectl expose`
