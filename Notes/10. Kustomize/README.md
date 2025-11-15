# CKA Kustomize Revision Notes

## Core Concepts

- **Base**: Original YAML files
- **Overlays**: Changes applied over base files

## Directory Structure Management

1.  Dry-run (See what will change)

```bash
kubectl diff -k overlays/staging/
```

2. Apply the configuration

```bash
kubectl apply -k overlays/staging/
```

3. View the final rendered YAML

```bash
kustomize build overlays/staging/
```

## Exam Tip:

The kustomization.yaml file is tiny.
You only need to remember 5 simple fields:

```bash
resources:
patchesStrategicMerge:
images:
commonLabels:
commonAnnotations:

```

### 🎯 Here is the strategy successful candidates use.

_STEP 1_: copy base YAML → use as patch
_STEP 2_: add minimal kustomization.yaml
_STEP 3_: test with kubectl kustomize .
_STEP 4_: apply with kubectl apply -k .

That’s it.

### Approach 1: Directory-based Resources

Each directory has its own `kustomization.yaml` file and Kustomize automatically scans it to create the resource:

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - db/
  - message-broker/
  - nginx/
```

### Approach 2: Direct File References

Add all files directly in the resource section:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - nginx-depl.yaml
  - nginx-service.yaml
```

## Transformers

[Documentation for more Transformers](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/#kustomize-feature-list)

### Common Labels

```yaml
commonLabels: # labels to add to all resources and selectors
```

### Namespaces

```yaml
namespaces: # Add namespace to each resource
```

### Image Updates

Original deployment:

```yaml
# web-dep.yaml
spec:
  container:
    - name: web
      image: nginx:1.2.3
```

#### Update Image Name

```yaml
# kustomization.yaml
images:
  - name: nginx # name of the old image
    newName: redis # name of new image
```

#### Update Image Version

```yaml
# kustomization.yaml
images:
  - name: nginx # name of the image
    newTag: "1.2.4" # New tag
```

### Complete Transformer Solution

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# 1️⃣ Namespace transformer
namespace: demo-namespace

# 2️⃣ Common labels transformer
commonLabels:
  team: devops
  owner: parimal

# 3️⃣ Image transformer (updates the image tag or name)
images:
  - name: nginx
    newName: nginx
    newTag: "1.27"

# 4️⃣ Name prefix and suffix transformers
namePrefix: prod-
nameSuffix: -v1

# Resources
resources:
  - deployment.yaml
  - service.yaml
```

Other common transformers: `commonAnnotations`, `namePrefix`, `nameSuffix`

## Patches

To create a patch, 3 parameters must be provided (there are more):

### Patch Parameters

- **Operation Type:**
  - add
  - remove
  - replace
- **Target Resource:** resource the patch needs to be applied on
  - Kind
  - Version/Group
  - Name
  - Namespace
  - labels selector
  - annotation selector
- **Value:** What is the value that will either be replaced or added with?

**📚 NOTE: For detailed notes on visit [Strategic Merge vs JSON 🔧](./README2.md)**

### JSON Patch Examples

#### Update Name

```yaml
# kustomization.yaml
patches:
  - target:
      kind: Deployment
      name: nginx-deployment
    patch: |-
      - op: replace
        path: /metadata/name
        value: web-deployment
```

#### Update Replica Count

```yaml
# kustomization.yaml
patches:
  - target:
      kind: Deployment
      name: nginx-deployment
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 5
```

### Strategic Merge Method

```yaml
# kustomization.yaml
patches:
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
          name: nginx-deployment
      spec:
          replicas: 5
```

## Working with Dictionary (Strategic Merge)

### Original Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: my-app
        component: api
    spec:
      containers:
        - name: my-app
          image: nginx:1.25
```

### Using External Patch File

```yaml
# kustomization.yaml - Strategic Merge
resources:
  - deployment.yaml

patches:
  - path: patch-component.yaml
```

You can use inline (as in previous example) or use separate file.

### Replace Field

```yaml
# patch-component.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    metadata:
      labels:
        component: web
```

### Add New Field

```yaml
# patch-component.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    metadata:
      labels:
        org: example.com
```

This will add new label along with other original `component: web`

### Remove Field

```yaml
# patch-component.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    metadata:
      labels:
        org: null
```

## Working with Lists (Strategic Merge)

### Original Deployment with Container

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: my-app
        component: api
    spec:
      containers:
        - name: nginx
          image: nginx
```

### Replace Container

```yaml
# patch-component.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
        - name: nginx
          image: nginx
```

### Add Container (This will merge two containers)

```yaml
# patch-component.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
        - name: redis
          image: redis
```

### ⚡Remove Container (Remove the redis we added above)

```yaml
# patch-component.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  template:
    spec:
      containers:
        - $patch: delete # IMP
          name: redis
```

## Patches Vs Transformers

### Kustomize Patches

- **Purpose**: Fine-tune or override fields in specific Kubernetes resources
- **Think of it as**: Editing a single file in a folder without touching the others
- **Example scenario**: You have a Deployment for nginx and want to set replicas to 3 and change the container port — only for that Deployment

### Kustomize Transformers

- **Purpose**: Apply bulk, rule-based modifications to multiple resources automatically
- **Think of it as**: Doing a "find and replace" across the entire project
- **Example scenario**: Add a team=backend label, set all resources to namespace dev, and change every image tag to v2

### 💡 Rule of Thumb

- **Use patches** when you know exactly what you want to change and where
- **Use transformers** when you want to apply a systematic change across many manifests

## Overlay

Kustomize is a Kubernetes configuration management tool that lets you customize YAML manifests without templates. It follows a base + overlay pattern for managing configurations across different environments.

## Key Commands

### Dry Run & Preview

```bash
# Preview changes before applying
kubectl diff -k overlays/staging/

# View final generated YAML without applying
kustomize build overlays/staging/
```

### Apply Changes

```bash
# Apply kustomization to cluster
kubectl apply -k overlays/staging/
```

### Edit Operations

```bash
# Add resources to kustomization
kustomize edit add resource file.yaml

# Add patches to kustomization
kustomize edit add patch patch.yaml
```

## Directory Structure

```
kustomize/
├── base/
│   ├── deployment.yaml        # Base Deployment manifest
│   ├── service.yaml           # Base Service manifest
│   └── kustomization.yaml     # References the base resources
│
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml # References base & applies dev-specific changes
    │   └── patch.yaml         # Patches for dev environment
    │
    ├── staging/
    │   ├── kustomization.yaml # References base & applies staging changes
    │   └── patch.yaml         # Patches for staging environment
    │
    └── prod/
        ├── kustomization.yaml # References base & applies prod changes
        └── patch.yaml         # Patches for production environment
```

## Base Configuration

### Base kustomization.yaml

```yaml
# base/kustomization.yaml
resources:
  - deployment.yaml
  - service.yaml
```

## Overlay Configuration

### Overlay kustomization.yaml

```yaml
# overlays/staging/kustomization.yaml
resources:
  - ../../base

patchesStrategicMerge:
  - patch.yaml
```

### Patch Example

```yaml
# overlays/staging/patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: myapp
          image: myimage:staging
```

## Workflow Examples

### 1. Development Workflow

```bash
# 1. Preview changes
kubectl diff -k overlays/dev/

# 2. Apply to dev environment
kubectl apply -k overlays/dev/

# 3. Verify deployment
kubectl get pods -l app=myapp
```

### 2. Staging Deployment

```bash
# 1. Check final manifest
kustomize build overlays/staging/

# 2. Apply to staging
kubectl apply -k overlays/staging/
```

### 3. Production Deployment

```bash
# 1. Dry run first
kubectl diff -k overlays/prod/

# 2. Apply to production
kubectl apply -k overlays/prod/
```

## Important Gotchas ⚠️

### Label/Selector Immutability

- **DO NOT** change `spec.selector` after deployment creation
- `spec.selector` is an immutable field in Deployments
- Apply environment labels only to `template.metadata.labels` to avoid errors

```yaml
# ❌ WRONG - Don't change spec.selector
spec:
  selector:
    matchLabels:
      app: myapp
      env: staging  # This will cause immutable field error

# ✅ CORRECT - Only change template labels
spec:
  selector:
    matchLabels:
      app: myapp  # Keep selector unchanged
  template:
    metadata:
      labels:
        app: myapp
        env: staging  # Safe to change template labels
```

### 3. Resource Management

```bash
# Add new resources incrementally
kustomize edit add resource configmap.yaml
kustomize edit add resource secret.yaml
```

### 4. Validation Workflow

```bash
# Always validate before applying
kustomize build overlays/prod/ | kubectl apply --dry-run=client -f -

# Then apply
kubectl apply -k overlays/prod/
```

## Componenets

## What is a Component?

A reusable piece of Kustomize configuration that can be mixed into multiple kustomizations.

## Basic Component Structure

### Component Definition (`monitoring/kustomization.yaml`)

```yaml
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

resources:
  - prometheus.yaml
  - grafana.yaml

patches:
  - path: add-monitoring-labels.yaml
    target:
      kind: Deployment
```

### Using Components (`main/kustomization.yaml`)

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - app.yaml

components:
  - ../monitoring
  - ../security
```

## Quick Examples

### 1. Monitoring Component

```yaml
# components/monitoring/kustomization.yaml
kind: Component
resources:
  - serviceMonitor.yaml
patches:
  - patch: |-
      - op: add
        path: /metadata/annotations/prometheus.io~1scrape
        value: "true"
    target:
      kind: Service
```

### 2. Security Component

```yaml
# components/security/kustomization.yaml
kind: Component
patches:
  - patch: |-
      - op: add
        path: /spec/template/spec/securityContext
        value:
          runAsNonRoot: true
          runAsUser: 1000
    target:
      kind: Deployment
```

### 3. Using Multiple Components

```yaml
# overlays/prod/kustomization.yaml
kind: Kustomization
resources:
  - ../../base

components:
  - ../../components/monitoring
  - ../../components/security
  - ../../components/backup
```

## Component vs Base/Overlay

- **Component**: Optional, reusable addon
- **Base**: Core configuration
- **Overlay**: Environment-specific changes

## Key Gotchas ⚠️

### 1. Component Order Matters

```yaml
components:
  - ../security # Applied first
  - ../monitoring # Applied second (can override security)
```

### 2. No Resources in Component Root

❌ **Wrong**: Put resources directly in component kustomization

```yaml
kind: Component
resources:
  - deployment.yaml
patches:
  - patch: |-
      # This patch might conflict with the deployment above
```

✅ **Better**: Keep components focused on patches/additions

### 3. Namespace Handling

Components don't automatically inherit namespace from main kustomization:

```yaml
# Component needs explicit namespace if required
kind: Component
namespace: monitoring # Add this if component resources need specific namespace
```

### 4. Path References

Components use relative paths from their own location:

```yaml
# In components/monitoring/kustomization.yaml
resources:
  - ./prometheus.yaml # Relative to component directory
  - ../shared/config.yaml # Not relative to main kustomization
```

### 5. Transformer Conflicts

Multiple components applying same transformation can conflict:

```yaml
# Component A adds label: app=foo
# Component B adds label: app=bar
# Result: Only last one wins (app=bar)
```

## Common Patterns

### Environment-Specific Components

```yaml
# staging/kustomization.yaml
components:
- ../components/debug
- ../components/low-resources

# prod/kustomization.yaml
components:
- ../components/monitoring
- ../components/security
- ../components/backup
```

### Feature Flags

```yaml
# Enable optional features
components:
  - ../components/redis # Only if caching needed
  - ../components/metrics # Only if monitoring needed
```

### Conditional Inclusion

```bash
# Use environment variable to conditionally include
kustomize build . --enable-alpha-plugins \
  $([ "$ENABLE_MONITORING" = "true" ] && echo "--component ../monitoring")
```

## Testing Components

```bash
# Test component in isolation
cd components/monitoring
kustomize build .

# Test with main kustomization
cd main
kustomize build .
```

## Best Practices

- Keep components small and focused
- Use descriptive component names
- Test components independently
- Document component dependencies
- Avoid circular dependencies between components

## Common Use Cases

### 1. Environment-Specific Images

```yaml
# Base uses generic image
image: myapp:latest

# Staging overlay patches with specific tag
image: myapp:v1.2.3-staging

# Prod overlay patches with stable tag
image: myapp:v1.2.3
```

### 2. Replica Scaling

```yaml
# Base has minimal replicas
replicas: 1

# Staging increases replicas
replicas: 2

# Production scales up
replicas: 5
```

### 3. Resource Limits

```yaml
# Staging patch
resources:
  limits:
    memory: "512Mi"
    cpu: "500m"

# Production patch
resources:
  limits:
    memory: "2Gi"
    cpu: "1000m"
```

## Troubleshooting

### Common Issues

1. **Immutable field errors**: Check if you're modifying selector labels
2. **Resource not found**: Verify base path in overlay kustomization
3. **Patch not applying**: Ensure patch targets correct resource name/namespace

### Debug Commands

```bash
# View final configuration
kustomize build overlays/staging/

# Validate syntax
kustomize build overlays/staging/ | kubectl apply --dry-run=client -f -

# Check differences
kubectl diff -k overlays/staging/
```

## Summary

- Use `kustomize build` to preview final YAML
- Use `kubectl diff -k` for dry runs
- Use `kubectl apply -k` to deploy
- Keep base minimal, customize in overlays
- Be careful with immutable fields like `spec.selector`

## Quick Commands

```bash
# Preview changes
kubectl kustomize .

# Apply directly
kubectl apply -k .

# Generate and save
kubectl kustomize . > output.yaml
```
