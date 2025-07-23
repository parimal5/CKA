<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Application Lifecycle Management</h3>
</div>

## Rolling Updates and the RollingUpdate Strategy

A Rolling Update is the default update strategy for Deployments. Instead of terminating all old Pods and starting all new ones at once (which can cause downtime), Kubernetes updates Pods incrementally, ensuring some Pods are always available during the process.

#### 🛠️ RollingUpdate Strategy Parameters

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 1 # Number or percentage of Pods that can be unavailable during the update.
    maxSurge: 1 # Number or percentage of extra Pods (above desired count) that can be created temporarily during the update.
```

#### Use Case Example:

You're deploying a new version of your app (v2), replacing the old one (v1):

```bash
kubectl set image deployment my-app my-container=my-app:v2
```

> This triggers a rolling update based on the above strategy.

#### 🧯Rollback Support

```bash
kubectl rollout undo deployment my-app
```

#### 📋 Checking Rolling Update Status

```bash
kubectl rollout status deployment my-app
```

### 📦 Other Strategies: Recreate Strategy:

Terminates all old Pods first, then creates new ones.

```yaml
strategy:
  type: Recreate
```

## Commands and Arguments

<img src="image.png" alt="alt text" width="600" height="300"/>

### Summary

| Docker Concept | Kubernetes Field | Notes                         |
| -------------- | ---------------- | ----------------------------- |
| `ENTRYPOINT`   | `command`        | Overrides entrypoint in image |
| `CMD`          | `args`           | Passed as args to entrypoint  |

## Configuring Environment Variables

Direct assignment of key-value pairs in Pod/Deployment spec:

```yaml
env:
  - name: ENVIRONMENT
    value: "production"
  - name: DEBUG_MODE
    value: "false"
```

**Key Points:**

- Simple static values
- Hardcoded in manifest
- Good for non-sensitive, unchanging data

## 2. ConfigMap

### Imperative Commands

```bash
# Create from literals
kubectl create configmap app-config --from-literal=ENVIRONMENT=production --from-literal=DEBUG_MODE=false

# Create from file
kubectl create configmap app-config --from-file=config.properties

# Create from directory
kubectl create configmap app-config --from-file=config-dir/
```

### Usage in Pod

```yaml
# Load all key-value pairs using envFrom
envFrom:
  - configMapRef:
      name: my-config

# OR Load specific key using env
env:
  - name: ENVIRONMENT
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: ENVIRONMENT
```

**Key Points:**

- Non-sensitive configuration data
- Can be updated without rebuilding image
- Supports key-value pairs and files

## 3. Secret

### Imperative Commands

```bash
# Create from literals
kubectl create secret generic app-secret --from-literal=DB_PASSWORD=mysecret --from-literal=API_KEY=abc123

# Create from file
kubectl create secret generic app-secret --from-file=credentials.txt

# Create TLS secret
kubectl create secret tls tls-secret --cert=path/to/cert.crt --key=path/to/key.key

# Create docker-registry secret
kubectl create secret docker-registry regcred --docker-server=myregistry.com --docker-username=user --docker-password=pass
```

#### Types

- `generic` - Arbitrary user-defined data
- `docker-registry` - Docker registry credentials
- `tls` - TLS certificates and keys
- `service-account-token` - Service account tokens

### Usage in Pod

```yaml
# Load all keys with envFrom
envFrom:
  - secretRef:
      name: app-secret

# OR Load individual key with env
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_PASSWORD
```

**Key Points:**

- Base64 encoded sensitive data
- Passwords, tokens, certificates
- Not encrypted at rest by default (use encryption at rest)
- Limited to 1MB per Secret
- **Declarative approach requires manual base64 encoding:** `echo -n 'password' | base64`
