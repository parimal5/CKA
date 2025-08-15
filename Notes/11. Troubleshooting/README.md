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

## Worker Node Failures

### Initial Node Status Check

```bash
kubectl get nodes
kubectl describe node worker-01
```

### System Health Verification

Check basic system resources using Linux commands:

```bash
df -hT          # Disk usage
free -h         # Memory usage
top             # CPU utilization
```

### Kubelet Service Status

```bash
sudo systemctl status kubelet
sudo journalctl -u kubelet -f
```

### Certificate Validation

```bash
sudo openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout
```

> ⚡**NOTE**: Make sure it is not expired, and is been issued by proper CA., they are part of right group `system:node`

## Critical Configuration Files

### 1. Kubelet Connection Config

**Path**: `/etc/kubernetes/kubelet.conf`

If the kublet is not able to connect with api server or cluster look in this file.

Check for:

- Ensure the API server IP/port is correct
- Ensure CA and client certs exist (base64 data is present)
- If the IP is a hostname (e.g., controlplane), make sure /etc/hosts resolves it

💡 **If this file is wrong, kubelet can't even register with the cluster.**

### 2. Kubelet Runtime Config

**Path**: `/var/lib/kubelet/config.yaml`

This is the KubeletConfiguration file.

Verify:

- Paths for `tlsCertFile`, `tlsPrivateKeyFile`
- Check `authentication` and `authorization` settings
- Look for wrong DNS settings (`clusterDNS`, `clusterDomain`)
- This is a kubeconfig file used by kubelet to talk to the API server

💡 **If this file is wrong, kubelet may run but won't manage pods correctly.**

## API Server Connectivity Test

```bash
curl -k https://<control-plane-ip>:6443
```

## ✅ Troubleshooting Order Summary

1. **Logs** → `journalctl -u kubelet`
2. **Connection config** → `/etc/kubernetes/kubelet.conf`
3. **Runtime config** → `/var/lib/kubelet/config.yaml`
4. **Certificates** → `/var/lib/kubelet/pki/`
5. **Startup flags** → `/etc/systemd/system/kubelet.service.d/`
6. **API reachability** → `curl test`
