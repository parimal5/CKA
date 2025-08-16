### 4-Step DNS Troubleshooting Flow

1️⃣ Check CoreDNS is running

```bash
kubectl get pods -n kube-system

kubectl logs -n kube-system
```

2️⃣Check Pod’s DNS config

- Are we even talking to the right DNS server?

```bash
kubectl exec -it <pod> -- cat /etc/resolv.conf
```

- ✅ nameserver matches kube-dns Service IP
- ❌ If wrong → Fix kubelet --cluster-dns flag (`/var/lib/kubelet/config.yaml`)

3️⃣ Check kube-dns Service

```bash
kubectl get svc kube-dns -n kube-system
```

4️⃣ Check if service records exist

Is CoreDNS giving the right answers?

```bash
kubectl run tmp --rm -it --image=busybox:1.28 -- nslookup kubernetes.default
```

✅ Works → DNS fine
❌ NXDOMAIN → missing service or CoreDNS config missing cluster.local zone

### CoreDNS Pods in Pending State

👉 Cause: This usually happens when no CNI (Container Network Interface) plugin is installed or configured properly.

- If CNI is missing → install/verify your CNI plugin (Flannel, Calico, Weave, etc.).

### CoreDNS Pods in CrashLoopBackOff or Error

👉 Cause: SELinux + older Docker issue

```bash
kubectl edit pod coredns -n kube-system
```

```yaml
securityContext:
  allowPrivilegeEscalation: false # Set this to true
```

### Loop detection in CoreDNS

👉 Cause: The host node’s /etc/resolv.conf points back to itself, creating a DNS resolution loop.

Tell kubelet to use the real resolv.conf:

```yaml
resolvConf: /run/systemd/resolve/resolv.conf
```

> Add this to you `/var/lib/kublet/config.yaml`

### 🔎 3. kube-dns Service has No Endpoints

👉 Even if CoreDNS Pods are running, if the Service selector is wrong, or Pods aren’t healthy, Service endpoints will be empty.

Check:

```bash
kubectl -n kube-system get svc kube-dns
kubectl -n kube-system get ep kube-dns
```

- If no endpoints → the Service can’t forward traffic → DNS fails.
- Fix: Make sure the Service selectors (k8s-app=kube-dns) match CoreDNS Pods’ labels.
