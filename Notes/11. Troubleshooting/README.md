<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Troubleshooting</h3>
</div>

## Application Failure Troubleshooting

### Issue: Users unable to access application

### 1. Check Web Server Connectivity

```bash
curl http://web-service-ip:node-port
```

### 2. Verify Service Endpoint Discovery

Check if service has discovered pod `endpoints`:

```bash
kubectl describe svc web-service
kubectl get deploy --show-labels
```

**If no endpoints found, verify selectors/labels match between service and pods.**

### 3. Validate Port Configuration

Ensure service `targetPort` matches pod's `containerPort`.

### 4. Check Pod Status

Verify pod is running and not restarting:

```bash
kubectl get pods
kubectl describe pod web-pod-name
```

### 5. Examine Logs

For current pod:

```bash
kubectl logs web-pod-name
kubectl logs web-pod-name -f  # follow logs
```

🔴 **Important**: If pod is restarting, check previous pod logs for actual error:

```bash
kubectl logs web-pod-name --previous
```
