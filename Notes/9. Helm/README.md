<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Helm</h3>
</div>

## 🔹 Helm Release

A deployed instance of a Helm chart.

```bash
helm install my-app bitnami/nginx     # Create release
helm list                             # List releases
helm uninstall my-app                 # Delete release
```

---

## 🔹 Helm Revisions

Each upgrade/rollback = new revision (starts at 1).

```bash
helm history my-app                   # View revision history
helm rollback my-app 2                # Roll back to revision 2
```

---

## 🔹 Helm Repository

HTTP/HTTPS based chart storage.

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update                      # Update repo index
helm search repo nginx               # Search charts
helm repo list                       # List repos
```

---

## 🔹 Essential Chart Commands

```bash
helm create mychart                   # Create new chart
helm lint mychart                     # Validate chart
helm template mychart                 # Render templates (dry-run)
helm upgrade my-app ./mychart         # Upgrade release
helm show values bitnami/nginx        # Show default values
```

---

## 🔹 Common Operations

```bash
# Install with custom values
helm install my-app bitnami/nginx -f values.yaml
helm install my-app bitnami/nginx --set service.type=LoadBalancer

# Upgrade or install (if not exists)
helm upgrade --install my-app ./mychart

# Check status
helm status my-app
helm get values my-app

# Dry run before install
helm install my-app ./mychart --dry-run
```

---

## 🔹 Most Used Flags

- `--dry-run` - Test without deploying
- `-f values.yaml` - Use custom values file
- `--set key=value` - Override single value
- `-n namespace` - Specify namespace

---

## 🔹 Quick Workflow

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install my-nginx bitnami/nginx
helm list
helm upgrade my-nginx bitnami/nginx --set replicaCount=3
helm rollback my-nginx 1
helm uninstall my-nginx
```
