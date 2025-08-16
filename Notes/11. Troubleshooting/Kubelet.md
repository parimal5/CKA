## Kubelet Troubleshooting

IMP:

```bash
cat /var/lib/kubelet/config.yaml
cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
ls /var/lib/kubelet/pki/
openssl x509 -in /var/lib/kubelet/pki/kubelet-client-current.pem -text -noout
```

### Service not running

- `systemctl` not started, disabled, or failed.
- `journalctl -u kubelet -f` to check the logs
- `systemctl restart kubelet`
- `systemctl enable kubelet`

### Configuration issues

- Wrong config file `(/var/lib/kubelet/config.yaml)`.
- Wrong flags in service unit file (`/etc/systemd/system/kubelet.service.d/10-kubeadm.conf`).
- Wrong `--resolv-conf`, `--network-plugin`, or `--kubeconfig`.

### Certificates

Expired/missing `/var/lib/kubelet/pki/kubelet-client.crt`.

### Node Registration Issues

Kubelet can’t talk to API server (connection refused).

- Check kublet-apiserver config file at `/etc/kubernetes/kubelet.conf`

Make sure the port is corrent and kind-control-plane resolve to IP().

server: https://kind-control-plane:6443
