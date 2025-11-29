## Question Bank and Notes

### Q1. Create a deployment name `web-app` with nginx:1.16 and then update the deploy to version nginx:1.17 and Ensure the version update is recorded in resource annotations.

Solution:

Create deployement

```bash
kubectl create deployment nginx-deploy --image=nginx:1.16 --replicas=1 --record
```

Upgrade deployment

```bash
kubectl set image deployment/nginx-deploy nginx=nginx:1.17 --record
```

Verify Deployment

```bash
kubectl rollout status deployment/nginx-deploy
```

Another way if `--record` got deprecated

```bash
kubectl set image deployment/mydeploy mycontainer=myimage:v2
kubectl annotate deployment/mydeploy kubernetes.io/change-cause="Updated to v2" --overwrite

```

### Q2. Find the schedulable nodes ion the cluster and save the name and count in to the file. /root/nodes.txt

> NOTE: Always check if the file is given already or you ahve to create it.

Now the schedulable node are those that do not ahve taints effect `NoSchedule`

```bash
k get node -o yaml | grep taints -A 3
```

#### Q3. Create a deployment/DaemonSet with lable app=prod, tier=db, dept=software.

If question ask you to add labels or create object with label then you need to udpate the labels at all the places

```yaml
metadata.labels
spec.selector.matchLabels
spec.template.metadata.labels
```

Eg.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deploy
  labels: # <----------------HERE
    app: pod
    dept: it
    tier: db
spec:
  selector:
    matchLabels: # <----------------HERE
      app: pod
      dept: it
      tier: db
  template:
    metadata:
      labels: # <----------------HERE
        app: pod
        dept: it
        tier: db
    spec:
      containers:
        - name: httpd
          image: httpd:2.4-alpine
```
