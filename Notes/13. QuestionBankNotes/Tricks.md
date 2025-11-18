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

### API Server Issues

Check if the apiserver pod is running or exited state:

- Exited state: invalid manigest, wrong argument
- Runnnig: connection issue to other component like issue with port, ip, url etc.

- **API Server manifest yaml not correct (invalid yaml)**
  - In this cases you won't be able to see the logs or even the pod it self for the kubeapiserver. \
    - `/var/log/pods` # nothing
    - `crictl logs` # nothing
  - Solution Workflow:
  - use kubelet logs `journalctl -xeu kubelet | grep apiserver`
  - Error you will see `Could not process manifest file`
- **API Server with wrong flag/wrong argument**
  - Configure the Apiserver manifest with a new argument `--this-is-very-wrong` .
  - This time you will be able to find the pod using `crictl ps -a` but in exited state.
  - then use `crictl logs <pod-id>`
- **Misconfigure ETCD connection in API Server**
  - When you change the connection to etcd or any server eg: ` --etcd-servers=this-is-very-wrong` (instead --etcd-servers=127.0.0.1:2379)
  - Or you will get this error when the `IP` or `Port` for the ETCD is wrong.
  - In such situation the when you do `crictl ps` the apiseerver pod won't be in exited state like above sitaion it will be running
  - So when you chek the logs `crictl logs <pod-id>` you wiill get error as
    - `transport: Error while dialing: dial tcp 127.0.0.1:2379 operation was canceled`
      - If you see this error that means `127.0.0.1:2379` look at the port and you know this belong to etcd so its etcd issues.
      - And when its connection issue that means api server is not able to reach etcd.
      - Sometime you get error as below again same issue the etcd server ip or port is wrong.
    - `Error while dialing dial tcp: address this-is-very-wrong: missing port in address. Reconnecting...`
