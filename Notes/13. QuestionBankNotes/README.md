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

### Q3. Create a deployment/DaemonSet with lable app=prod, tier=db, dept=software.

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

### Q4: A `multi-container` pod is deployed in the default namespace with 2 contaienr name `c1` and `c2`. You have to get the logs and container id for the container named `c2` in the pod. And then restart the container `c2` and write the events to `root/events.log` file. Store logs in `logs.txt` and containerID in `containerID.txt`

**Step 1:** Grab the logs of the container c2

```bash
k logs multi-container -c c2 > logs.txt
```

**Step 2:** Grab the ID of the container

```bash
crictl ps

echo "<ID>" > containerID.txt
```

**Step 3:** Restart the container

```bash
crictl ps # Grab the ID of the container

crictl stop <ID> # Stop the container

crictl rm <ID> # Remove the container

# The kubelet will recreate that container.
```

**Step 4:** Store the Events

```bash
k get events --sort-by=.metadata.creationTimestamp > root/events.log
```

**Extra**: Sometime they ask you to store the events only for that pod instead of all the pod

```bash
k get events --sort-by=.metadata.creationTimestamp --field-selector involvedObject.name=multi-container
```

**Extra**: Sometime they will not ask you to store the event or logs but the commands you use to grab the logs or events so read the question carefully

### Q5: List all the pod in safari namespace and then sort by creation order.

```bash
k get pod -n safari --sort-by=.metadata.creationTimestamp
```

**Bonus:** If we need to print in reverse order or decending order the use `tac` command

```bash
k get pod -n safari --sort-by=.metadata.creationTimestamp | tac
```

**Bonus:** Sort the pod by `priority`

```bash
k get pod -n safari --sort-by=.spec.priority
```
