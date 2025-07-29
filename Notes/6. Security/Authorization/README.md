<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Authorization, RBAC, ABAC, Webhooks</h3>
</div>

## Authorization Modes

- **Node** - Node authorization
- **RBAC** - Role-Based Access Control
- **ABAC** - Attribute-Based Access Control
- **WebHooks** - External authorization (e.g., OPA)

**Check current modes:**

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
# Look for: --authorization-mode=Node,RBAC
```

## RBAC Components Overview

- **Role** - Namespaced permissions
- **RoleBinding** - Assigns Role to users/groups/service accounts
- **ClusterRole** - Cluster-wide permissions
- **ClusterRoleBinding** - Assigns ClusterRole to users/groups/service accounts

**Key Differences:**

- **Role/RoleBinding**: Namespaced resources (pods, deployments, jobs)
- **ClusterRole/ClusterRoleBinding**: Cluster-wide resources (nodes, PV, storage classes)

**Important:** ClusterRole can also be used for namespaced resources to grant access across ALL namespaces.

### View Existing Roles

```bash
# Namespaced roles
kubectl get role -n <namespace>
kubectl get rolebinding -n <namespace>

# Cluster roles
kubectl get clusterrole
kubectl get clusterrolebinding

# View details
kubectl get role <role-name> -n <namespace> -o yaml
kubectl get clusterrole <role-name> -o yaml
```

### Create Role (Namespaced)

```bash
kubectl create role developer \
  --verb=create,list,delete \
  --resource=pods,deployments
```

### Create RoleBinding (Namespaced)

```bash
kubectl create rolebinding dev-user-binding \
  --role=developer \
  --user=dev-user
```

### Create ClusterRole (Cluster-wide)

```bash
kubectl create clusterrole storage-admin \
  --verb=create,list,delete \
  --resource=nodes,persistentvolumes,storageclasses
```

### Create ClusterRoleBinding (Cluster-wide)

```bash
kubectl create clusterrolebinding storage-admin-binding \
  --clusterrole=storage-admin \
  --user=admin
```

> ⚡**NOTE**: Dry Run (Always Test First) : `--dry-run=client -o yaml`

### Validate Permissions

```bash
# Generic syntax
kubectl auth can-i <verb> <resource>

# Check if user can perform action
kubectl auth can-i get deployments -n blue --as dev-user
kubectl auth can-i create pods --as dev-user

# Alternative: Try actual commands
kubectl get pods --as dev-user
kubectl create deployment nginx --image=nginx --as dev-user
```

## Key Points

- **Role** = permissions within a namespace
- **ClusterRole** = permissions across entire cluster
- **RoleBinding** = assigns role to user/group/service account (namespaced)
- **ClusterRoleBinding** = assigns cluster role to user/group/service account (cluster-wide)
- Always use `--dry-run=client -o yaml` to verify before applying
- Use `kubectl auth can-i` to test permissions
- **Always use plural form for resources** (`pods`, `deployments`, not `pod`, `deployment`)
