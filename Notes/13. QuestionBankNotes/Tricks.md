### If you quickely need to recreate the static pods.

- use `sudo systemctl restart kubelet`

This will recreate all the static pods.

### One of the nodes is NotReady. Fix it.

FIRST thing you should do for Node NotReady:

```bash
kubectl describe node <node>
```

You MUST start here because the issue is almost always visible in the node conditions:

- NetworkUnavailable
- Kubelet stopped
- NotEnoughResources
- DiskPressure
- Certificate expired
- CNI issue
- etc

Then SSH into node:

```bash
systemctl status kubelet
journalctl -u kubelet -f

systemctl restart kubelet
```

🔑 Correct flow

1. kubectl describe node
2. Check conditions + events
3. SSH to node
4. Check kubelet
5. Logs
6. Fix root cause

### A pod named db is stuck in Pending. Fix it.

> 👉 If a pod is Pending, you MUST NOT go to logs — because there is no running container.

Correct CKA troubleshooting flow for Pending:

````bash
kubectl describe pod <name>  # must show Events
kubectl get events --sort-by=.lastTimestamp
kubectl get nodes -o wide
kubectl describe node```
````

Look for:

- FailedScheduling
- Unschedulable
- NodeSelector mismatch
- Resource limits exceeded
- No matching taint tolerations
- PVC not bound
- ImagePull secret missing
- NetworkPolicy blocking

### Rollout Stuck / Deployment Failing

#### Golden Flow for rollout stuck

```bash
kubectl rollout status deploy/api
kubectl describe deploy api
kubectl get rs
kubectl describe rs <name>
kubectl get pods
kubectl describe pod <name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

Common causes

- ❌ Wrong image
- ❌ ImagePullBackOff
- ❌ Readiness/Liveness probe failing
- ❌ Resource limits too small
- ❌ Bad command/args
- ❌ PVC not bound
- ❌ CrashLoopBackOff

Fix commands:

```bash
kubectl rollout undo deploy/api
```

```bash
kubectl rollout restart deploy/api
```

```bash
kubectl rollout undo deploy/api
```
