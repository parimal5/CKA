# CKA — Zero-BS Time-Saver README

**Purpose:** A compact, exam-oriented cheat sheet and troubleshooting README for the CKA. Focuses on non-obvious paths, quick commands, and step-by-step troubleshooting flows you should memorize before the exam.

> **Print/Save:** This file is intentionally terse and focused — use it as a quick reference inside the exam environment.

---

```bash
# note: -in file-name (order matters)
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text

helm show values <chart> | yq ea '.. | path | join(".")' | grep -i nodePort


```

# 1. Quick reference — MUST MEMORIZE

- **Static Pod manifests:** `/etc/kubernetes/manifests/`
- **kubelet config:** `/var/lib/kubelet/config.yaml` or `/etc/kubernetes/kubelet.conf`
- **kubelet systemd drop-in:** `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`
- **kubeadm cluster config:** `/etc/kubernetes/kubeadm-config.yaml`
- **etcd data dir:** `/var/lib/etcd`
- **etcd certs:** `/etc/kubernetes/pki/etcd`
- **cluster certs:** `/etc/kubernetes/pki`
- **admin kubeconfig:** `/etc/kubernetes/admin.conf`
- **user kubeconfig:** `~/.kube/config`

**Fast recall trick:** Control-plane files → `/etc/kubernetes/*` · Kubelet files → `/var/lib/kubelet/*` and systemd overrides.

---

# 2. Common ports (memorize)

| Component               | Default Port |
| ----------------------- | -----------: |
| kube-apiserver          |         6443 |
| etcd                    |         2379 |
| kubelet API             |        10250 |
| kube-scheduler          |        10251 |
| kube-controller-manager |        10252 |
| kube-proxy metrics      |        10249 |

---

# 3. Troubleshooting quick-playbook (order matters)

### Generic flow for ANY pod issue

1. `kubectl get po -A`
2. `kubectl describe po <pod> -n <ns>` → look at Events, Conditions, Node
3. `kubectl logs <pod> -n <ns>` (add `-c <container>` if multi-container)
4. If pod is crashlooping, use `kubectl logs --previous ...`
5. `kubectl get events --sort-by=.metadata.creationTimestamp`
6. `kubectl get po <pod> -o yaml` → inspect env, volumes, image, args

### Node NotReady / Node issues

1. `kubectl get nodes` → status
2. On node: `systemctl status kubelet` and `journalctl -xeu kubelet -n 200`
3. Check CRI: `crictl ps` or `docker ps` (depending on runtime)
4. `sudo crictl logs <container-id>` to inspect container-level issues
5. Disk pressure? `df -h` & `du -sh /var/lib/docker /var/lib/containerd /var/lib/kubelet`

### Control plane broken (apiserver / controller / scheduler)

1. `kubectl --kubeconfig=/etc/kubernetes/admin.conf get componentstatuses` (basic)
2. `ls /etc/kubernetes/manifests` — static pod manifests present?
3. `cat /etc/kubernetes/manifests/kube-apiserver.yaml` → args, certificates, ports
4. On control-plane node: `docker ps | grep kube` or `crictl ps` to see static pod containers
5. Check `/var/log` or `journalctl -xeu kubelet` for static pod creation errors

### kube-proxy issue

1. The config file for `kube-proxy` is stored as `configMap`.
2.

### etcd specific

- Health check:

```bash
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health
```

- Snapshot backup:

```bash
ETCDCTL_API=3 etcdctl snapshot save /opt/etcd-backup.db
```

### Certificate issues

- Inspect cert:

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```

- Common files to check:

```
/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/apiserver.crt
/etc/kubernetes/pki/apiserver-kubelet-client.crt
/etc/kubernetes/pki/front-proxy-ca.crt
/etc/kubernetes/pki/etcd/*.crt
```

---

# 4. Fast commands cheat (copy-paste ready)

- List all pods across namespaces: `kubectl get po -A`
- Describe quickly: `kubectl describe po <name> -n <ns>`
- Show events sorted: `kubectl get events --sort-by=.metadata.creationTimestamp`
- View yaml: `kubectl get po <name> -n <ns> -o yaml`
- Run ephemeral pod for network tests: `kubectl run -it --rm --image=busybox debug -- /bin/sh`
- Exec into pod: `kubectl exec -it <pod> -n <ns> -- /bin/sh`
- Copy file to pod: `kubectl cp file.txt <pod>:/tmp -n <ns>`
- Use specific kubeconfig: `kubectl --kubeconfig=/etc/kubernetes/admin.conf get nodes`
- API quick check: `curl -k https://127.0.0.1:6443/healthz`

---

# 5. Static pod recovery notes

- Kubelet watches `/etc/kubernetes/manifests/` for any `*.yaml` and creates static pod containers automatically.
- To force-recreate a static pod: fix YAML (or replace it) in that directory — kubelet will destroy & recreate.
- If kubelet can't create pod, check `journalctl -u kubelet` for manifest parsing errors or permissions.

---

# 6. Kubeconfig & context quicktips

- View contexts: `kubectl config get-contexts`
- Switch: `kubectl config use-context <ctx>`
- Inspect specific kubeconfig w/out switching: `kubectl --kubeconfig=/etc/kubernetes/admin.conf get componentstatuses`

---

# 7. Logs and systemd (node-level)

- Kubelet logs: `journalctl -u kubelet -f`
- Container runtime logs: `journalctl -u containerd -f` or `journalctl -u docker -f`
- If systemd units failed: `systemctl status kubelet --no-pager -l`

---

# 8. Non-obvious fast-hacks (time savers)

- Create quick pod yaml from run command: `kubectl run test --image=nginx --restart=Never -o yaml > pod.yaml`
- Dry-run apply locally: `kubectl apply -f file.yaml --dry-run=client`
- Edit live resource (use with caution): `kubectl edit pod <name> -n <ns>`
- Get previous logs for CrashLoopBackOff: `kubectl logs <pod> -n <ns> --previous`

---

# 9. Flashcard section (for mental drills)

- Q: Static pod path? → `/etc/kubernetes/manifests`
- Q: Kubelet config file? → `/var/lib/kubelet/config.yaml`
- Q: Admin kubeconfig path? → `/etc/kubernetes/admin.conf`
- Q: etcd default port? → `2379`
- Q: apiserver default port? → `6443`

---

# 10. Exam-day checklist (2-minute pre-check)

1. Confirm `kubectl` works: `kubectl get nodes`
2. Check cluster-wide events: `kubectl get events --sort-by=.metadata.creationTimestamp`
3. List pods in kube-system: `kubectl -n kube-system get po -o wide`
4. If any control-plane pods down: `ls /etc/kubernetes/manifests` → inspect YAMLs

---

# 11. Example scenarios (short)

### A. Pod unable to pull image

- `kubectl describe pod` → look for ImagePullBackOff or ErrImagePull → check image name, registry auth
- `kubectl get secrets -n <ns>` → imagePullSecrets
- Try pull on node: `docker pull <image>` or `crictl pull <image>`

### B. DNS failures inside cluster

- Exec into busybox: `kubectl run -it --rm --image=busybox dns-test -- nslookup kubernetes.default`
- Check CoreDNS pods: `kubectl -n kube-system get po -l k8s-app=kube-dns`
- Inspect ConfigMap: `kubectl -n kube-system get configmap coredns -o yaml`

---

# 12. Personal study tips (how to use this README)

- Memorize section 1 + ports + the 3 troubleshooting flows (pod / node / control-plane).
- Practice the commands until they are muscle memory; time yourself.
- During mock exams, force yourself to always run `kubectl get events --sort-by=.metadata.creationTimestamp` first.

---
