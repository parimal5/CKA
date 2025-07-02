# CKA Exam - Essential Imperative Commands

## Pod Management

### Create Pods
```bash
# Basic pod creation
kubectl run nginx --image=nginx

# Pod with specific port
kubectl run nginx --image=nginx --port=80

# Pod with environment variables
kubectl run nginx --image=nginx --env="VAR1=value1" --env="VAR2=value2"

# Pod with resource limits
kubectl run nginx --image=nginx --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'

# Pod with labels
kubectl run nginx --image=nginx --labels="app=web,env=prod"

# Pod with restart policy
kubectl run nginx --image=nginx --restart=Never

# Pod with command
kubectl run busybox --image=busybox --command -- sleep 3600

# Dry run to generate YAML
kubectl run nginx --image=nginx --dry-run=client -o yaml > pod.yaml
```

### Pod Operations
```bash
# Get pods
kubectl get pods
kubectl get pods -o wide
kubectl get pods --show-labels
kubectl get pods -l app=nginx

# Describe pod
kubectl describe pod nginx

# Delete pod
kubectl delete pod nginx
kubectl delete pod nginx --force --grace-period=0

# Execute commands in pod
kubectl exec nginx -- ls /
kubectl exec -it nginx -- /bin/bash

# Port forwarding
kubectl port-forward nginx 8080:80

# Logs
kubectl logs nginx
kubectl logs -f nginx
kubectl logs nginx --previous
```

## Deployment Management

### Create Deployments
```bash
# Basic deployment
kubectl create deployment nginx --image=nginx

# Deployment with replicas
kubectl create deployment nginx --image=nginx --replicas=3

# Deployment with port
kubectl create deployment nginx --image=nginx --port=80

# Generate YAML
kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > deployment.yaml
```

### Deployment Operations
```bash
# Scale deployment
kubectl scale deployment nginx --replicas=5

# Update deployment image
kubectl set image deployment/nginx nginx=nginx:1.21

# Rollout operations
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx
kubectl rollout undo deployment/nginx --to-revision=2

# Edit deployment
kubectl edit deployment nginx

# Delete deployment
kubectl delete deployment nginx
```

## Service Management

### Create Services
```bash
# ClusterIP service
kubectl expose pod nginx --port=80 --target-port=80

# NodePort service
kubectl expose pod nginx --port=80 --target-port=80 --type=NodePort

# LoadBalancer service
kubectl expose deployment nginx --port=80 --target-port=80 --type=LoadBalancer

# Service with specific selector
kubectl expose deployment nginx --port=80 --target-port=80 --selector="app=nginx"

# Generate service YAML
kubectl expose pod nginx --port=80 --dry-run=client -o yaml > service.yaml

# Create service imperatively
kubectl create service clusterip nginx --tcp=80:80
kubectl create service nodeport nginx --tcp=80:80
```

## ConfigMap and Secret Management

### ConfigMaps
```bash
# Create configmap from literal
kubectl create configmap app-config --from-literal=key1=value1 --from-literal=key2=value2

# Create configmap from file
kubectl create configmap app-config --from-file=config.properties

# Create configmap from directory
kubectl create configmap app-config --from-file=config-dir/

# Generate YAML
kubectl create configmap app-config --from-literal=key1=value1 --dry-run=client -o yaml > configmap.yaml
```

### Secrets
```bash
# Create generic secret
kubectl create secret generic app-secret --from-literal=username=admin --from-literal=password=secret

# Create secret from file
kubectl create secret generic app-secret --from-file=credentials.txt

# Create TLS secret
kubectl create secret tls tls-secret --cert=path/to/cert --key=path/to/key

# Create docker registry secret
kubectl create secret docker-registry regcred --docker-server=myregistry.io --docker-username=user --docker-password=pass --docker-email=email@example.com
```

## Namespace Management

```bash
# Create namespace
kubectl create namespace dev

# Set default namespace
kubectl config set-context --current --namespace=dev

# List all resources in namespace
kubectl get all -n dev

# Delete namespace
kubectl delete namespace dev
```

## Resource Management and Troubleshooting

### Get Resources
```bash
# Get all resources
kubectl get all
kubectl get all -A  # All namespaces

# Get specific resources
kubectl get nodes
kubectl get pods -A
kubectl get svc -A
kubectl get deploy -A
kubectl get pv,pvc

# Get with output formats
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl get pods -o json
kubectl get pods -o jsonpath='{.items[*].metadata.name}'
```

### Describe and Debug
```bash
# Describe resources
kubectl describe node node1
kubectl describe pod nginx
kubectl describe svc nginx

# Debug commands
kubectl get events --sort-by='.lastTimestamp'
kubectl top nodes
kubectl top pods

# Resource usage
kubectl describe node node1 | grep -A 5 "Allocated resources"
```

## RBAC Commands

```bash
# Create service account
kubectl create serviceaccount sa-name

# Create role
kubectl create role pod-reader --verb=get,list,watch --resource=pods

# Create cluster role
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods

# Create role binding
kubectl create rolebinding pod-reader-binding --role=pod-reader --user=jane

# Create cluster role binding
kubectl create clusterrolebinding pod-reader-binding --clusterrole=pod-reader --user=jane

# Check permissions
kubectl auth can-i create pods
kubectl auth can-i create pods --as=jane
```

## Job and CronJob Management

```bash
# Create job
kubectl create job hello --image=busybox -- echo "Hello World"

# Create job from cronjob
kubectl create job hello-manual --from=cronjob/hello

# Create cronjob
kubectl create cronjob hello --schedule="*/1 * * * *" --image=busybox -- echo "Hello World"
```

## Utility Commands

### Context and Config
```bash
# View contexts
kubectl config get-contexts

# Switch context
kubectl config use-context context-name

# Set namespace
kubectl config set-context --current --namespace=namespace-name
```

### Labels and Annotations
```bash
# Add label
kubectl label pod nginx app=web

# Remove label
kubectl label pod nginx app-

# Add annotation
kubectl annotate pod nginx description="My nginx pod"

# Remove annotation
kubectl annotate pod nginx description-
```

### Patch and Edit
```bash
# Patch resource
kubectl patch pod nginx -p '{"spec":{"containers":[{"name":"nginx","image":"nginx:1.21"}]}}'

# Edit resource
kubectl edit pod nginx
kubectl edit deployment nginx
```

## Important Flags and Options

### Common Flags
- `--dry-run=client -o yaml`: Generate YAML without creating resource
- `--force --grace-period=0`: Force delete immediately
- `-A` or `--all-namespaces`: Show resources from all namespaces
- `-l` or `--selector`: Filter by labels
- `-o wide`: Show additional information
- `--show-labels`: Display labels
- `-w` or `--watch`: Watch for changes

### Output Formats
- `-o yaml`: YAML format
- `-o json`: JSON format
- `-o wide`: Additional columns
- `-o name`: Resource names only
- `-o jsonpath`: Custom output using JSONPath

## Exam Tips

1. **Always use `--dry-run=client -o yaml`** to generate manifests quickly
2. **Use short forms**: `po` for pods, `svc` for services, `deploy` for deployments, `ns` for namespaces
3. **Remember the `--force --grace-period=0`** for immediate deletion when needed
4. **Use `kubectl explain`** to understand resource specifications: `kubectl explain pod.spec`
5. **Practice with `kubectl run`** and `kubectl create` commands as they're faster than writing YAML from scratch
6. **Use `kubectl get events --sort-by='.lastTimestamp'`** for troubleshooting
7. **Remember to check current context and namespace** before running commands

## Quick Reference Commands for Exam

```bash
# Quick pod with sleep for testing
kubectl run test --image=busybox --command -- sleep 3600

# Quick nginx deployment
kubectl create deployment nginx --image=nginx --replicas=3

# Quick service exposure
kubectl expose deployment nginx --port=80 --type=NodePort

# Quick configmap
kubectl create configmap config --from-literal=key=value

# Quick secret
kubectl create secret generic secret --from-literal=password=mypass

# Quick job
kubectl create job test --image=busybox -- echo "test"
```
