# kubectl JSONPath Query Guide

A comprehensive guide for querying Kubernetes resources using JSONPath expressions with kubectl.

## Table of Contents

- [JSONPath Basics](#jsonpath-basics)
- [Basic Syntax Examples](#basic-syntax-examples)
- [Advanced Filtering](#advanced-filtering)
- [kubectl JSONPath Usage](#kubectl-jsonpath-usage)
- [Common Use Cases](#common-use-cases)

## JSONPath Basics

JSONPath allows you to extract specific data from JSON structures. Here's a sample Kubernetes Pod manifest we'll use for examples:

```json
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      },
      {
        "image": "redis:alpine",
        "name": "redis-container"
      }
    ],
    "nodeName": "node01"
  }
}
```

## Basic Syntax Examples

### Simple Field Access

| JSONPath Expression       | Result        | Description             |
| ------------------------- | ------------- | ----------------------- |
| `kind`                    | `"Pod"`       | Get the kind field      |
| `metadata.name`           | `"nginx-pod"` | Get nested field        |
| `spec.containers[0].name` | `"nginx"`     | Get first array element |

### Array Operations

```jsonpath
# Get all container names
spec.containers[*].name

# Get first container's image
spec.containers[0].image
```

**Example with root-level array:**

```json
[{ "name": "John" }, { "name": "Steve" }, { "name": "Mark" }]

# For root-level arrays, use $
$[*].name
```

## Advanced Filtering

### Conditional Filtering

Use `?()` syntax for filtering based on conditions:

```json
# Get image for container named 'redis-container'

spec.containers[?(@.name=='redis-container')].image

# Result: "redis:alpine"
```

## kubectl JSONPath Usage

### Single Resource Queries

When querying a specific resource (single object):

```bash
# Get container image from specific pod
kubectl get pod nginx -o=jsonpath='{.spec.containers[0].image}'
```

**Output:** `nginx:1.21`

### Multiple Resource Queries

When querying multiple resources, use `.items[]` to access the array:

```bash
# Get first pod's first container image
kubectl get pods -o=jsonpath='{.items[0].spec.containers[0].image}'

# Get all container images from all pods
kubectl get pods -o=jsonpath='{.items[*].spec.containers[*].image}'

# Conditional query across multiple pods
kubectl get pods -o=jsonpath='{.items[0].spec.containers[?(@.name=="redis-container")].image}'
```

## Common Use Cases

### Resource Information

```bash
# Get all pod names
kubectl get pods -o=jsonpath='{.items[*].metadata.name}'

# Get pod names and their node assignments
kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.nodeName}{"\n"}{end}'

# Get all container images in use
kubectl get pods -o=jsonpath='{.items[*].spec.containers[*].image}' | tr ' ' '\n' | sort | uniq
```

### Status and Conditions

```bash
# Get pod phases
kubectl get pods -o=jsonpath='{.items[*].status.phase}'

# Get ready status for all containers
kubectl get pods -o=jsonpath='{.items[*].status.containerStatuses[*].ready}'

# Get pods not in Running state
kubectl get pods -o=jsonpath='{.items[?(@.status.phase!="Running")].metadata.name}'
```

### Resource Specifications

```bash
# Get CPU requests for all containers
kubectl get pods -o=jsonpath='{.items[*].spec.containers[*].resources.requests.cpu}'

# Get environment variables for specific container
kubectl get pods -o=jsonpath='{.items[*].spec.containers[?(@.name=="app")].env[*].name}'

# Get service account names
kubectl get pods -o=jsonpath='{.items[*].spec.serviceAccountName}'
```

## Tips and Best Practices

1. **Use single quotes** around JSONPath expressions in bash to avoid shell interpretation
2. **Test expressions** on single resources before applying to multiple resources
3. **Use `.items[*]`** when working with resource lists (get pods, get services, etc.)
4. **Combine with other tools** like `tr`, `sort`, and `uniq` for better output formatting
5. **Use `{range}` syntax** for formatted output with multiple fields

### Formatting Output

```bash
# Custom formatted output
kubectl get pods -o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.status.phase}{"\n"}{end}'

# JSON output with specific fields only
kubectl get pods -o=jsonpath='{.items[*].metadata.name}' | jq -R -s 'split(" ") | map(select(length > 0))'
```

---

**Note:** JSONPath expressions in kubectl must be enclosed in curly braces `{}` and the entire expression should be quoted to prevent shell interpretation.
