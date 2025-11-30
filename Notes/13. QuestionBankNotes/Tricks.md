# Keep in mind:

- Always validate your results. If you create pod make sure it get created correctely and running and in correct namespace.
- Sometime they ask you to store the result in some file like logs, eventsm, yaml, or sometime they ask to store the command you use to get that result, So always read the question carefully.
- When you write the the command in exam you will use shortcut as `k` and if the quesiton ask you to store the command in a file then make sure you do not store the command with alias `k` store full command using `kubectl`

Shortcut

```bash
export do="--dry-run=client -oyaml"

# Usage
k run $do nas --image nginx
```

### kubelet - If you quickely need to recreate the static pods.

- use `sudo systemctl restart kubelet`

This will recreate all the static pods.

- ⚡The certification path of kubelet is

```bash
/var/lib/kubelet/pki/kubelet-client-current.pem
```

If the kubelet is failing with a cert error, FIRST check:

### One of the nodes is NotReady. Fix it.

```bash
cat /etc/kubernetes/kubelet.conf

# Make sure that theses are pointint to correct variable
client-certificate: /var/lib/kubelet/pki/kubelet-client-current.pem
client-key: /var/lib/kubelet/pki/kubelet-client-current.pem

```

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

kubectl get pod <pod-name> -oyaml | yq  # ⚡At the end of the manifest file you will see the detailed reson for the pod failure as comapre to describe
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

### `--command`

- This flag only work with pod created using `run` command
- Do not work with `create` command.
- `kubectl create deployment -- sleep 3500`
- Use `sh -c` when you have to give multiple commands
- `kubectl create deployment -- sh -c echo "Parimal" && sleep 10`

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

### Service Account Role and RoleBiding

If the question mention to give `read-access` then use `--verb=get,list,watch`

### Network Policy

- If you need to `deny all ingress or egreess` rule. Just do not specify any rule in yaml and it will create the polilcy that denyt everything.
- in Network Policy you ahve to select podSelector

```yaml
spec:
  podSelector: # Target --> Who this policy is applied to.
    matchLabels:
      role: db
  policyTypes:
    - Ingress
  ingress:
    - from: # Which Pod can send ingress traffic to our target.
        - podSelector:
            matchLabels:
              role: frontend
        - namespaceSelector: # Which Namespace can send ingress traffic to our target.
            matchLabels:
              project: myproject
```

- ⚡Important: If you need target as complete `namespace` instead of the `pod` as in above example

```yaml
spec:
  podSelector: {} # This will target all the resources in the namespace.
```

### ETCD Backup and Restore

- You already have commannd in docs for back and restore.
- Workflow for restoring the db:
  - No need to stop `api-server`
  - `--data-dir=/var/lib/etcd` is where we need to resotre that data but make sure you either move out the data from this path or rename it to something else.

```bash
mv /var/lib/etcd /var/lib/etcd.bak
mkdir /var/lib/etcd

# OR you can delte and recreate too.
```

- Afte doing this then you are ready to restore the backup

### Broken kube-config

Exam Statement

```txt
The default kubeconfig (~/.kube/config) is misconfigured.
Use the admin kubeconfig located at /etc/kubernetes/admin.conf to complete this task.”

# OR

User developer cannot list pods in kube-system.
Use the admin kubeconfig to investigate and fix the RBAC issue.
```

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```

### Role and Role Binding and Service Account

```bash
k auth can-i get svc --as=system:serviceaccount:default:dev
```

### Schedule Pod on Specific Node

```bash
nodeName: node01
```

### Cluster Upgrade

If you have to downgrade the cluster like kubelet version then just running the

```bash
# Current: 1.34.1 --> 1.32.1
sudo apt-get install -y kubelet='1.32.1-*' kubectl='1.32.1-*'
```

⚡This will not work until you use the flag `--allow-downgrade`

## Rolling Update - Strategy `Recreate`

```yaml
strategy:
  type: Recreate
```

### How to run a DNS Test in from POD

```bash
kubectl exec -it mypod -- nslookup kubernetes.default

or

kubectl run netshoot --image=busybox --rm -it -- nslookup kubernetes.default
```

## Kubelet

### Issue 1

How do you know the kublet is down?

```bash
k get node #  Check if the node is in NotReady State
```

```bash
systemctl status kublet
```

```bash
journalctl -xeu kubelet
```

```bash
find / | grep kubeadm

# This will give you a file
/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# This file contain the info about the kubelet server(used by systemctl)
```

```bash
# controlplane:~$ cat /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

Base on the logs you can look though thises errors in differnet files.

### Issue 2

Sometime you see the node is not ready and you ssh to that node and see the kubelet is not running and you did systemctl start kubelet but and again chek the status but still the kubelet is not up and running.

Below are the kubelet log you seee the message `203/EXEC` that means in the `/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf` file the EXEC command has wrong path

```bash
Nov 28 14:51:33 controlplane systemd[1]: kubelet.service: Main process exited, code=exited, status=203/EXEC
░░ Subject: Unit process exited
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░
░░ An ExecStart= process belonging to unit kubelet.service has exited.
░░
░░ The process' exit code is 'exited' and its exit status is 203.
```

```bash
# here the path should be /usr/bin/kubelet/
ExecStart=/usr/bin/locals/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS
```

### Issue 3

If logs show “unknown flag” or parsing error
Problem: Environment files (flags)

```bash
/var/lib/kubelet/kubeadm-flags.env
/etc/default/local/kubelet
```

## Pod Related Issues

- Pod stuck in pending state
  - check if the `nodeName` is specified on the pod or deployment

## Logs

- When you pull logs from `Pod` you can simple use command

```bash
kubectl logs <pod> -n namespace --all-container # for multiple contianers
```

But what if you need to get logs from deployment

```bash
kubectl logs deploy/<deploy-name>
```

> So you need to mention the type deploy/nginx-deploy

## Ingress

- Ingress create the NodePort for you at port 30080 or 30443 to expose you ingree resource to internet
- so you can use the <IP>:30080 to reach to the ingress resoruce
- then ingress will use the service of type ClusterIP to internally sent that traffic to the backend PODS.
- `http://world.universe.mine:30080/europe/`
- In the above example
- `host`: `world.universe.mine:30080` # Don't get confused by this, think of this as complete URL.
- `path`: `europe`
- `service:port`: `<clusterIP-service>:80`
- `rule`: `--rule=world.universe.mine:30080/europe:europe:80`
- `world.universe.mine` --> This get resolve to nodePort IP when you add the entery in `/etc/hosts` file.
- Make sure to use `pathType: Prefix` not `Exact` in ingress resource.

## Network Policy

```bash
ports:
- port: 53
  protocol: TCP
- port: 53
  protocol: UDP
```

## PodAffininity and PodAntiAffinity

- Suppose you ahve exisintg workload on POD A running on node01 (cloud be scheduled aby any means nodeSelector, or randomSchedule etc).
- Now you need to deploy some new work and your reqirement is that I want my new workload/pod to be scheduled on the same node where my Pod A is placed.
- Now to achive this senerio we use `PodAffininity`
- Eg. Logging pods near the application pods

- Same when you need to schedule the workload away from the exising pod then use `PodAntiAffinity`

## Cluster Setup/Upgrade/Install/Initialize

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors=NumCPU,Mem --kubernetes-version=1.34.1
```

- Always check first if kubeadm, kubelet, kubectl version

### Node join command

```bash
kubeadm token create --print-join-command
```

**OUTPUT**

```bash
kubeadm join 172.30.1.2:6443 --token 85ykxz.u04jyejt0ijbgpja --discovery-token-ca-cert-hash sha256:df0c776e06xxxxxxxx
```

### Read out certificate expiration

```bash
kubeadm certs check-expiration
```

### Renew the certs

```bash
kubeadm certs renew scheduler.conf
oR
kubeadm certs renew apiserver
```

## PortForwarding

```bash
kubectl port-forward svc/nginx-service 8080:80
```

## ⚡HPA

This is the imperative command to creat ethe HPA resource

```bash
kubectl autoscale deployment hpa-demo --cpu=50% --min=1 --max=5
```

## CronJob

Shortcut Memory Trick (works in exam)

```bash
min hour day month weekday
```

If something should run:

- every minute → `put *`
- every X minutes → `*/X`
- every hour → put `minute=0`
- daily → `minute=0, hour=0`

### 🎯 Examples EXACTLY like CKA asks

#### ✔ “Run every 2 minutes”

```bash
*/2 * * * *`
```

#### ✔ “Run daily”

```bash
0 0 * * *`
```

#### ✔ “Run every hour”

```bash
0 * * * *`
```

#### ✔ “Run every minute”

```bash
* * * * *`
```

### LimitRange/ResourceQuota --> Admission Controllers

_LimitRange and ResourceQuota policies are enforced by Admission Controllers. So sotime in question they ask you to enfore Admission Controllers instead of saying apply limit to pod or namespace etc._

- If the question says “total CPU/memory for the namespace” → use ResourceQuota.
- If the question says “limit or default resources for each pod/container” → use LimitRange.

**ResourceQuota**: all the pod in the namespace can use max of 500m cpu there is no fix liimit which pod should use whateither one pod can use 400m and remming 100m bno matter but most they can use is 500m

**LimitRange**: each pod has limit set what max they can use like 200m so each pod cannot cross taht 200m cpu limit
