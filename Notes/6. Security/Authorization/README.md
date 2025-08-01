<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Authorization, RBAC, Service Accounts, Network Policies</h3>
</div>

## Authorization Modes:

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

### Key Points

- **Role** = permissions within a namespace
- **ClusterRole** = permissions across entire cluster
- **RoleBinding** = assigns role to user/group/service account (namespaced)
- **ClusterRoleBinding** = assigns cluster role to user/group/service account (cluster-wide)
- Always use `--dry-run=client -o yaml` to verify before applying
- Use `kubectl auth can-i` to test permissions
- **Always use plural form for resources** (`pods`, `deployments`, not `pod`, `deployment`)

## Service Account Operations

### View Service Accounts

```bash
kubectl get sa
kubectl describe sa <name>
```

### Create Service Account

```bash
kubectl create sa <my-service-account>
```

### Assign to Pod/Deployment

```yaml
spec:
  serviceAccount: my-service-account
```

### Create Token

```bash
kubectl create token <name-of-service-account>
```

### Complete RBAC Workflow

### 1. Create Role

```bash
kubectl create role my-role --verb=get,list --resource=pod -n dev
```

### 2. Create Service Account and Token

```bash
kubectl create sa my-sa -n dev
kubectl create token my-sa
```

### 3. Create Role Binding

```bash
kubectl create rolebinding my-role-rb -n dev --serviceaccount=dev:my-sa --role=my-role
```

### 4. Assign to Deployment/Pod

```bash
# Edit deployment
kubectl edit deployment <name>

# Add to spec:
spec:
  template:
    spec:
      serviceAccount: my-sa
```

**Note**: For pods, deletion and recreation required - edit not supported.

## Security Context

### 🔐 Where can securityContext be applied?

- Pod-level: applies to all containers in the pod.
- Container-level: overrides pod-level settings for individual containers.

```yaml
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
    - name: app
      image: nginx
      securityContext:
        allowPrivilegeEscalation: false
        privileged: false
        readOnlyRootFilesystem: true
        runAsNonRoot: true
```

---

When you don’t specify a securityContext in your Kubernetes manifests, Kubernetes simply doesn’t enforce any security constraints for user ID, privileges, or filesystem behavior.

- Containers may run as root
- Privilege escalation allowed
- Unrestricted Linux capabilities
- Writable root filesystem

---

When you define securityContext at both the Pod and Container levels

- Kubernetes prioritizes container-level securityContext over the pod-level one for any overlapping fields.

```yaml
spec:
  securityContext:
    runAsUser: 1000
  containers:
    securityContext:
      runAsUser: 1111 # Overrides pod-level runAsUser
```

---

Some option are POD Level and Some are Container:
| Field | Scope | Description |
|---------------------------|-------------|-----------------------------------------------------------------------------|
| runAsUser | Pod/Container | UID to run the process as. Container-level overrides Pod-level. |
| runAsGroup | Pod/Container | GID to run the process as. Container-level overrides Pod-level. |
| runAsNonRoot | Pod/Container | Ensures the container does not run as root. |
| fsGroup | Pod | Group ID for all volumes; useful for shared storage access. |
| allowPrivilegeEscalation | Container | Prevents processes from gaining more privileges (e.g., via `sudo`). |
| privileged | Container | Gives container full host access. Avoid unless absolutely needed. |
| readOnlyRootFilesystem | Container | Makes root filesystem read-only. Good for hardening. |
| capabilities | Container | Fine-grained control over Linux capabilities (add/drop specific ones). |

## NetworkPolicy

A Kubernetes resource that controls traffic flow (Ingress/Egress) at IP/port level for pods in a namespace.

> 🧠 **Key Point**: By default, all traffic is allowed between pods. Once NetworkPolicy is applied, pods become isolated and only allow defined traffic.

### Basic Structure

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: example-policy
spec:
  podSelector:
    matchLabels:
      app: database # Selects pods this policy applies to
  policyTypes:
    - Ingress # Control incoming traffic
    - Egress # Control outgoing traffic
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 3306
  egress:
    - to:
        - ipBlock:
            cidr: 192.168.1.0/24
      ports:
        - protocol: TCP
          port: 5432
```

### Example Scenario

**Setup**:

- Internal pod (frontend)
- DB pod (database)
- External backup server: `192.172.52.10`

**Requirement**:

- Only internal pod can access DB pod
- DB pod can send data to backup server

### Solution Breakdown

#### 1. Target Pod Selection

```yaml
podSelector:
  matchLabels:
    app: database # Apply policy to DB pods
```

#### 2. Allow Incoming Traffic (Ingress)

```yaml
ingress:
  - from:
      - podSelector:
          matchLabels:
            app: internal # Only internal pods can access
    ports:
      - protocol: TCP
        port: 3306 # MySQL port
```

#### 3. Allow Outgoing Traffic (Egress)

```yaml
egress:
  - to:
      - ipBlock:
          cidr: 192.172.52.10/32 # Specific backup server
    ports:
      - protocol: TCP
        port: 5978
```

### Key Rules to Remember

#### Selector Types

- `podSelector`: Select pods by labels
- `namespaceSelector`: Select entire namespaces
- `ipBlock`: Select IP ranges (external traffic)

#### Traffic Direction

- `ingress`: Incoming traffic TO the selected pods
- `egress`: Outgoing traffic FROM the selected pods

### Important Notes

⚡ **Stateful Traffic**: NetworkPolicies are stateful - if incoming traffic is allowed, the response is automatically allowed back.

🎯 **Think from Pod's Perspective**: Always consider from the viewpoint of the pod whose traffic you're controlling.

### **Can I use either Ingress or Egress to achieve the same scenario?**

Yes, in most cases, you can achieve the same effect using either:

- Ingress policy on the target pods , or
- Egress policy on the source pod

| Policy Type | Controls...                      | Use When...                                                  |
| ----------- | -------------------------------- | ------------------------------------------------------------ |
| **Ingress** | Who can **access** the pod       | You want to protect a sensitive service (`db`, `payroll`)    |
| **Egress**  | Where a pod can **send** traffic | You want to limit a pod's reach or control data exfiltration |

Example: Apply to db to allow traffic to db only from internal-app

```yaml
# This says "Only internal-app is allowed to reach db"
podSelector: name: db
ingress:
  from:
    - podSelector: name: internal-app

```

Example: Apply to internal-app to allow traffic from internal-app only to db and payroll

```yaml
# This says "internal-app can only talk to db:3306 and payroll:8080"
podSelector: name: internal-app
egress:
  - to:
      - podSelector: name: db
    ports:
      - port: 3306
  - to:
      - podSelector: name: payroll
    ports:
      - port: 8080
```

#### Common Commands

```bash
# List network policies
kubectl get networkpolicy
kubectl get netpol

# List in specific namespace
kubectl get netpol -n <namespace>

# Describe policy details
kubectl describe netpol <policy-name> -n <namespace>

# Delete network policy
kubectl delete netpol <policy-name> -n <namespace>
```

### Quick Reference Patterns

#### Allow All Ingress

```yaml
ingress:
  - {}
```

#### Allow All Egress

```yaml
egress:
  - {}
```

#### Block All Traffic

```yaml
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

#### Allow from Same Namespace

```yaml
ingress:
  - from:
      - podSelector: {}
```

---

## Custom Resource Definition

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: <spec.names.plural + "." + spec.group>  # e.g., "datacenters.traffic.controller"
spec:
  group: <your-group-name>  # e.g., "traffic.controller"
  versions:
    - name: <version-name>  # e.g., "v1"
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                <field-name-1>:
                  type: <data-type>  # e.g., string, integer, boolean
                <field-name-2>:
                  type: <data-type>
                ...
  scope: <Namespaced|Cluster>  # Choose "Namespaced" or "Cluster"
  names:
    plural: <plural-form>       # e.g., "datacenters"
    singular: <singular-form>   # e.g., "datacenter"
    kind: <Kind>                # e.g., "Global"
    shortNames:
      - <short-name>            # e.g., "glb"
```

## Custom Resource

```yaml
apiVersion: <spec.group>/<version-name>  # e.g., "traffic.controller/v1"
kind: <Kind>  # Must match spec.names.kind
metadata:
  name: <custom-resource-name>  # e.g., "datacenter"
spec:
  <field-name-1>: <value-of-correct-type>  # Match the schema defined in the CRD
  <field-name-2>: <value-of-correct-type>
  ...
```
