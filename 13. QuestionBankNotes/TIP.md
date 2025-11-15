# CKA Troubleshooting Cheat Sheet

Quick, practical README for troubleshooting Kubernetes problems in the CKA exam. Designed to be scanned fast under pressure — keep this open while you practice.

---

## Purpose

A compact, exam-focused workflow and checklist to identify and fix common Kubernetes failures quickly. Memorize the _pattern_, not every command.

---

## Troubleshooting Mindset

When something breaks, it will almost always fall into one of these buckets:

- Pod / Application issue
- Node / Kubelet issue
- Control-plane component issue (apiserver, controller-manager, scheduler)
- Etcd issue
- Networking / DNS / CNI issue
- RBAC / Auth issue
- Certificates / TLS issue

Start by running `kubectl get nodes` and `kubectl get pods -A` — these two commands most often point you to the right category.

---

## Fast Start Commands (first things to run)

```bash
kubectl get nodes
kubectl get pods -A
kubectl cluster-info
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> [-c container] -n <ns>
```

If you suspect node or control-plane: SSH into node and check

```bash
systemctl status kubelet
journalctl -u kubelet -xe --no-pager
ls /etc/kubernetes/manifests
cat /etc/kubernetes/manifests/<component>.yaml
```

---

## Step-by-step Framework

### 1) Identify the _category_

Use `kubectl get nodes` and `kubectl get pods -A`.

- Node NotReady → kubelet/runtime
- System pods failing → CNI/CoreDNS/control plane
- Single namespace → app-level issue

### 2) Control plane issues (static pods)

Static pods live in `/etc/kubernetes/manifests/` on control-plane nodes.

Files to check:

- `/etc/kubernetes/manifests/kube-apiserver.yaml`
- `/etc/kubernetes/manifests/kube-controller-manager.yaml`
- `/etc/kubernetes/manifests/kube-scheduler.yaml`
- `/etc/kubernetes/manifests/etcd.yaml`

Look for: wrong IPs/ports, `--etcd-servers`, `--kubeconfig`, cert file paths, wrong `advertise-address`.

### 3) Etcd checks

Common problems: wrong endpoints, cert mismatch, listen/advertise URLs.

Commands:

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health
```

Check `etcd.yaml` for `--listen-client-urls` and `--advertise-client-urls` and confirm IPs.

### 4) Kubelet issues

Check service and logs on the node:

```bash
systemctl status kubelet
journalctl -u kubelet -xe --no-pager
cat /var/lib/kubelet/config.yaml
```

Look for: bad kubeconfig path, expired certs, cgroup driver mismatch, wrong node IP.

### 5) Pod troubleshooting (app-level)

```bash
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns>
kubectl get events -n <ns>
```

Check events for image pull errors, scheduling failures, PVC binds, probe failures, missing secrets/configmaps.

### 6) DNS / CNI

If pods can’t resolve or network traffic fails:

```bash
kubectl get pods -n kube-system | grep coredns
kubectl logs -n kube-system coredns-<id>
kubectl get ds -n kube-system
```

Check CNI plugin pods (flannel/calico/weave) and node network config (`ip addr`, `iptables -L`).

### 7) Certificates

If control-plane or API access fails, inspect `/etc/kubernetes/pki/` and cert-related flags in manifests. Look for expired certs or SAN mismatches.

---

## Quick Field Scan (open file and jump to these)

When you open a manifest, jump to lines with these flags and fields:

- `--etcd-servers`
- `--advertise-address`
- `--bind-address`
- `--kubeconfig`
- `--client-ca-file`, `--tls-cert-file`, `--tls-private-key-file`
- `volumes:` paths (missing file mounts)

You don’t need to read the whole file — just verify these critical values.

---

## Common Exam Fixes (examples)

- **API server wrong etcd URL** → fix `--etcd-servers` in `kube-apiserver.yaml`.
- **etcd not reachable** → correct listen/advertise URLs or cert paths in `etcd.yaml`.
- **kubelet failing** → check kubelet kubeconfig, restart kubelet, fix cgroup settings.
- **CoreDNS crashLoopBackOff** → check ConfigMap `coredns` and upstream resolvers, inspect coredns logs.
- **Pod ImagePullBackOff** → check image name, registry access, imagePullSecrets.

---

## Speed Techniques (to save exam time)

- Always start with `kubectl get nodes` and `kubectl get pods -A`.
- Learn to scan manifests for the 6 critical flags above.
- Memorize where static pods live and what files to check for each control-plane component.
- Practice common scenarios until they feel reflexive (Killercoda/Killer.sh/Hands-on labs).
- Use `grep` inside manifests to find suspicious flags quickly: `grep -n "etcd-servers\|kubeconfig\|advertise-address" -n /etc/kubernetes/manifests/*`

---

## Practice Plan

- Solve 40–50 targeted failure scenarios.
- Time yourself: 10–15 minutes per complex troubleshooting problem initially; aim to get under 5–7 minutes with practice.

---

## Final Notes

Troubleshooting is pattern recognition. The more failure cases you practice, the faster you’ll recognize the problem category and the exact file/flag to check.

Good luck — and if you want, I can convert this README into a printable 1-page cheat-sheet or generate 20 practice scenarios with step-by-step solutions.
