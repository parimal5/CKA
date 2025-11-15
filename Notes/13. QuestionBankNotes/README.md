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
