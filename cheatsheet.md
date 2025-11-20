```bash
note: -in file-name (order matters)
```

```bash
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text
```

```bash
helm show values <chart> | yq ea '.. | path | join(".")' | grep -i nodePort
```

```bash
find / | grep kubeadm # kublet conf file find
```
