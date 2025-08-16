## ✨Control Plane failure

- Check Node Status

```bash
kubectl get nodes
```

- Check Pod in the `kube-system` namespace status.**[If the componenet are deployed as PODs]**

For Pod-Based Components

```bash
kubectl get pods -n kube-system

# If kubectl fails → switch to docker/crictl directly:
docker ps | grep kube-apiserver
crictl ps | grep kube-apiserver
```

- If the component are deployed as service then check status using

For Service-Based Components

```bash
sudo systemctl status kube-apiserver
sudo systemctl status kube-controller-manager
sudo systemctl status kube-scheduler
sudo systemctl status kubelet
sudo systemctl status kube-proxy
sudo systemctl status etcd
```

- Check Logs for the POD base componenet

```bash
kuebctl logs <component-master> -n kubesystem

docker logs container_id
crictl logs container_id
```

Service-Based Component Logs

```bash
sudo journalctl -u kube-apiserver
sudo journalctl -u kube-scheduler -f
```

### Critical File Paths

Static Pod Manifests:

```
/etc/kubernetes/manifests/
├── kube-apiserver.yaml
├── kube-controller-manager.yaml
├── kube-scheduler.yaml
└── etcd.yaml
```

Certificates & Keys

```
/etc/kubernetes/pki/
├── ca.crt
├── ca.key
├── apiserver.crt
├── apiserver.key
├── apiserver-kubelet-client.crt
├── apiserver-kubelet-client.key
├── front-proxy-ca.crt
├── front-proxy-client.crt
├── etcd/
│   ├── ca.crt
│   ├── server.crt
│   └── server.key
└── sa.key, sa.pub
```

Configuration Files:

```
/etc/kubernetes/
├── admin.conf
├── controller-manager.conf
├── kubelet.conf
└── scheduler.conf
```

**NOTE:The Certificates are mounted as `Volumes` in the PODs.**

### Validate static pod manifests

```bash
ls -l /etc/kubernetes/manifests/
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

### Certification's Validation

The key ones:

- `/etc/kubernetes/pki/ca.crt` (cluster CA)
- `/etc/kubernetes/pki/apiserver.crt`
- `/etc/kubernetes/pki/apiserver-kubelet-client.crt`
- `/var/lib/kubelet/pki/kubelet-client-current.pem`

Some are certs (.crt or .pem) → these you can check with openssl x509.
