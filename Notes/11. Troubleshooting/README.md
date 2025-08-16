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

## Networking Failures
