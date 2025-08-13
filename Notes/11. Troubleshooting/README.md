<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Troubleshooting</h3>
</div>

### IMPORTANT Commands:

```bash
source <(kubectl completion bash) # set up autocomplete in bash into the current shell, bash-completion package should be installed first.
echo "source <(kubectl completion bash)" >> ~/.bashrc # add autocomplete permanently to your bash shell.
```

Change the default namespace

```bash
kubectl config set-context --current --namespace=<name>
```

## Application Failure Troubleshooting

### Issue: Users unable to access application

### 1. Check Web Server Connectivity

```bash
curl http://web-service-ip:node-port
```

### 2. Verify Service Endpoint Discovery

Check if service has discovered pod `endpoints`:

```bash
kubectl describe svc web-service
kubectl get deploy --show-labels
```

**If no endpoints found, verify selectors/labels match between service and pods.**

### 3. Validate Port Configuration

Ensure service `targetPort` matches pod's `containerPort`.

### 4. Check Pod Status

Verify pod is running and not restarting:

```bash
kubectl get pods
kubectl describe pod web-pod-name
```

### 5. Examine Logs

For current pod:

```bash
kubectl logs web-pod-name
kubectl logs web-pod-name -f  # follow logs
```

🔴 **Important**: If pod is restarting, check previous pod logs for actual error:

```bash
kubectl logs web-pod-name --previous
```

## Control Plane failure

- Check Node Status

```bash
kubectl get nodes
```

- Check Pod in the `kube-system` namespace status.**[If the componenet are deployed as PODs]**

For Pod-Based Components

```bash
kubectl get pods -n kube-system
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

- The Certificates are mounted as `Volumes` in the PODs.
