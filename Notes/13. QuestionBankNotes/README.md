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

### Q4: A `multi-container` pod is deployed in the default namespace with 2 contaienr name `c1` and `c2`. You have to get the logs and container id for the container named `c2` in the pod. And then restart the container `c2` and write the events to `root/events.log` file. Store logs in `logs.txt` and containerID in `containerID.txt`.

**Solution:**

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

**Solution:**

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

### Q6: The pod name util-pod is running in the default namespace, delete the pod without any delay (force delete) and copt the command in /root/command.txt.

**Solution:**

```bash
kubectl delete pod util-pod -n default --grace-period-0 --force
```

### Q6: There are pod running with label env=prod and env=dev. your task is to change the labels for all the pod with label env=dev to env=prod.

**Solution:**

```bash
k label pod my-pod env=prod --overwrite
```

### Q7. You need to setup a namespace for application team but make sure they should not be able to crete mopre than 5 pods in the namespace

**Solution:**

For this quesiton use `ResourceQuota`

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-quota
  namespace: some-namespace
spec:
  hard:
    requests.cpu: "2" # sum of all CPU requests in namespace ≤ 2 cores
    requests.memory: "4Gi" # sum of all memory requests ≤ 4Gi
    limits.cpu: "4" # sum of all CPU limits ≤ 4 cores
    limits.memory: "8Gi" # sum of all memory limits ≤ 8Gi
    pods: "10" # total pods in namespace ≤ 10
    persistentvolumeclaims: "5" # total PVCs ≤ 5
    # ... other resources / object counts or storage quotas ...
```

> If the ResourceQuota is create for cpu or memory and you try to create a pod witout specifing any resource request or limit the k8s will deny the request.

### Q8. Create a Helm chart named `nginx-chart` that deployes an Nginx application using the `nginx:1.24.0` image. Configure the deployment to run with `2 replicas`. Set the memory `request to 300Mi` and the memeory `limit to 500Mi`. Once the chart is created install it with the release name `my-nginx` in the `nginx-helm` namespace.

**Solution:**

This will create a sample chart in the directory you will get basic chart with nginx. Now fo this question you ahve to udpate the Values.yaml with the above values

```bash
helm create nginx-chart
```

```bash
vim nginx-chart/values.yml
```

Once all the changes are done create the namespace nginx-helm and install the with the name my-nginx

```bash
helm install my-nginx nginx-chart -n nginx-helm
```

**Extra: Sometime they deploy the helm chart for you and ask you to update something onthe exisitng chart**
So change the values.yaml with new changes

```bash
helm upgrade my-nginx nginx-chart -n nginx-helm
```

**Extra: Or sometime they ask you to de the rollback of that helm chart**

```bash
helm rollback my-nginx 1 -n nginx-helm
```

1 is the revison number we need to rollback to you can get the revision number using below command

```bash
helm histroy my-nginx
```

### Q9.In the `kube-prod` namespace configure pod admission controllers to enfore the default resource `request` and `limit` for container The conditions: In absence of CPU/memory request/limit will default to cpu 200m and memory 100Mi for request cpu 400m and 200Mi memory for limit

> So even though it says pod admission controllers but don't get confused with AdmissionControllers resource in k8s the limitRange or ResourceQuota and many other are know as AdmissionContrl enforser's

**Extra: There is another admission controllers Enforcer called `Pod Security Admission`**

_PSA applies at the namespace level_

```yaml
# MODE must be one of `enforce`, `audit`, or `warn`.
# LEVEL must be one of `privileged`, `baseline`, or `restricted`.
pod-security.kubernetes.io/<MODE>: <LEVEL>
```

For every PSA question in the CKA exam, the solution is basically:

- ✅ Add (or modify) the correct PSA `label on the namespace`.

```bash
kubectl label ns <ns> pod-security.kubernetes.io/enforce=restricted
```

### Q10: Deploy A CNI of you choice (the links are given for different CNI)

Best Option to choose: Flannel and Calico.

But there is sometime twist where the cluster default Pod Cidr Range is differne than the CNI you install so you need to update that default cidr range in CNI.

how do you know your cluster POD IP Range - Either `logs` for the CNI PODs or check the `apiServer manifest`

1. Flannel - Edit the ConfigMap (Recommended 1st Choice)

```bash
kubectl -n kube-flannel get configmap

kubectl -n kube-flannel edit configmap kube-flannel-cfg

```

```json
net-conf.json: |
  {
    "Network": "10.244.0.0/16",  # Match this to your current pod range
    "Backend": {
      "Type": "vxlan"
    }
  }
```

> Make sure after updating the config map you need to recreate the pod's (delete it it will get created automatically)

2. Calico - Edit the DaemonSet Env Variable

```bash
kubectl -n kube-system edit ds calico-node
```

```yaml
- name: CALICO_IPV4POOL_CIDR
  value: "192.168.0.0/16"
```

Change this value to match the cluster’s Pod CIDR.

### Q11. Create a secret with value passwd=mypass! and create a deployment that uses that secret as evnironment variable OR(add env variable to existing deployment)

The easiest way to add secret/configmap as environment variable to new deployment or exising deployment using imperative way is below without opeaning yaml

```bash
k set env deploy/test-dep --from='secret/mysec'
```

It will add all the secrets in you secret as env variables

```yaml
- env:
    - name: APITOKEN
      valueFrom:
        secretKeyRef:
          key: apitoken
          name: mysec
    - name: NAME
      valueFrom:
        secretKeyRef:
          key: name
          name: mysec
    - name: PASSWD
      valueFrom:
        secretKeyRef:
          key: passwd
          name: mysec
```

This will work only with deployment or ds etc not with standalone pod but you can do is

```bash
k set env pod/test-pod --from=secret/mysec --dry-run=client -oyaml > test.yml
```

```bash
k replace --force -f test.yml
```

### Q12.Add the following Hel repository with the name `nginx-repo` `https://kubernetesway.github.io/my-helm-repo`. Install the helm chart from `nginx-repo` with Release name: `example-app` and version: `0.1.0`. And then update the version to `0.2.0`

```bash
helm repo add nginx-repo  https://kubernetesway.github.io/my-helm-repo

helm repo list
```

Now we need to find out what charts are available in that repo

```bash
controlplane:~$ helm search repo nginx-repo

# NAME                    CHART VERSION   APP VERSION     DESCRIPTION
# nginx-repo/nginx-chart  0.2.0           1.25            A Helm chart for NGINX
```

This will only show you latest version

```bash
controlplane:~$ helm search repo nginx-repo --versions

# NAME                    CHART VERSION   APP VERSION     DESCRIPTION
# nginx-repo/nginx-chart  0.2.0           1.25            A Helm chart for NGINX
# nginx-repo/nginx-chart  0.1.0           1.21            A Helm chart for NGINX
```

Now you see all the available versions so now you can use the below command to install specific version:

```bash
helm install example-repo nginx-repo/nginx-chart --version=0.1.0
```

Now we need to upgrade it:

```bash
helm upgrade example-repo nginx-repo/nginx-chart --version=0.2.0
```

Verify using:

```bash
helm history example-repo
```

**Extra**

Now you need to deploy the helm chart with custom values.

First we need to get the values for the helm chart

```bash
helm show value nginx-repo/nginx-chart --version=0.2.0 > custom-values.yml
```

Now vim custom-values.yml and then update the required values you need

```bash
helm install example-repo nginx-repo/nginx-chart -f custom-values.yml --version=0.2.0
```

```bash
helm get manifest <REAEASE_NAME>
```

this will giove you final generated manifest so if you need to check which image the manifest is using then you can us then and then do grep image
