<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Logging and Monitoring</h3>
</div>

## Monitoring

### Deploy metric server using Github Repo

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

> This installs the latest version of Metrics Server in the kube-system namespace.

### Common Commands

```bash
kubectl top nodes
kubectl top node <node-name>
kubectl top pods --all-namespaces
kubectl top pod <pod-name> -n <namespace>
kubectl top pod <pod-name> -n <namespace>
kubectl top pod --sort-by=memory
kubectl top pod --sort-by=cpu
```

## Logging

### View Logs from a Single-Container Pod

```bash
kubectl logs <pod-name>
```

### View Logs from a Specific Container (if multiple containers in pod)

```bash
kubectl logs <pod-name> -c <container-name>
```

### If your pod restarted (e.g., due to a crash), use:

```bash
kubectl logs --previous <pod-name>
```

### View Logs for All Pods in a Deployment

```bash
kubectl logs -l app=<app-label>
```

### Stream Live Logs (like tail -f)

```bash
kubectl logs -f mypod -c mycontainer
```

> This will keep the terminal open and continuously print new log lines from the pod until you stop it (with Ctrl+C).
