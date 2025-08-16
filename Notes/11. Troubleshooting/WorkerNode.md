## ✨ Worker Node Failures

### Initial Node Status Check

```bash
kubectl get nodes
systemctl status kubelet
systemctl status containerd

journalctl -u kubelet

```

### Look for taints/conditions

```bash
kubectl describe node worker-01
```

### CNI Config File available

If directory is empty → Pods will be stuck in ContainerCreating.

```bash
ls -l /etc/cni/net.d/

# 10-flannel.conflist
# 10-calico.conflist
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
7. **CNI plugin configuration files** → `ls -l /etc/cni/net.d/10-flannel.conflist` (OR `10-calico.conflist`)
   `
