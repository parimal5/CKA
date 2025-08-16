## Kube-Proxy Traoubleshooting

The Kube-Proxy runs as Daemon-Set.

### Check kube-proxy status

```bash
kubectl get ds -n kube-system -o wide | grep kube-proxy

kubectl describe ds -n kube-system <kube-proxy-pod>

kubectl logs -n kube-system <kube-proxy-pod>

```

### Kube-Proxy Config File

```bash
/var/lib/kube-proxy/kubeconfig.conf
```

🔴**IMPORTANT NOTE**🔴

This is the path to the config file on the host.

When you see the ConfigMap for the kube-proxy you see the file it will be generete is called `config.conf` and not kubeconfig.conf

So that is the reason when you see the yaml for the Daemonset for kubeproxy

```yaml
# k get -n kube-system ds kube-proxy -o yaml
containers:
  - command:
      - /usr/local/bin/kube-proxy
      - --config=/var/lib/kube-proxy/config.conf ## The path for the file the config map create not the host path.
      - --hostname-override=$(NODE_NAME)
```
