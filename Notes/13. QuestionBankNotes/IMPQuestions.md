## Most Important Questions for CKA

### Q1. You have a frontend and backend deployment running in theior respective namespace. The 2 deployment need to communicate with each other. Your task is to apply an appropriate Network policyfrom the provided directory `~/netpol`.** NOTE: Do no delete or modify any existing network policy.**

**Solution**

Things to keep in mind:

- Like in AWS where the deny policy alwas take precedence thats not the case in K8S.
- Do not delete or modi the deny-all policy - currentely the front end and backend namespace have deny-all ingress and egress policy.
- here you do not have to create the policy the policy are given in the dir netpol/ and find the correct policy to apply. (also you need to make sure the policy should be least previliged)

- Now depending on the question you need to make sure that the fronend pod should allow the egress tioo backend and then backed shoudl allow the ingress from frontend.

- Now finally, after you did everying still the connection is broken, but why?
- Since both the namespace has deny all egress rule that means the pod cannot reach the CoreDNS service to get the IP resolve.
- So you might need to allow the egress connection from both namespace to kube-system namespace on prot 53 and protocol UDP (no need for TCP to same some time).

```yaml
- to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
  ports:
    - protocol: UDP
      port: 53
```

Command to verify:

```bash
k exec -n frontend -it my-fronted-pod -- curl -m 5 http://service.namespace:port
```

`-m 5` will timeout in 5 sec so you won't waste time in CKA

### Q2.ETCD Backup and Restore

When you do the restore and if the question ask you to resore in diffenret data dir like `/var/lib/etcd-backup` then you need to update the etcd.yaml to this new data-dir, so only thing you will change is the volume

### Q3 Join the Worker Node.

```bash
kubeadm token create --print-join-command
```

### Create a new user called john. Grant him access to the cluster using a `csr` named `john-developer`. Create a role developer which should grant John the permission to `create, list, get, update and delete pods` in the `development` namespace . The private key exists in the location: `/root/CKA/john`.key and csr at `/root/CKA/john.csr`. Important Note: As of kubernetes 1.19, the `CertificateSigningRequest` object expects a `signerName`.

Please refer to the documentation to see an example. The documentation tab is available at.

[Issue a Certificate for a Kubernetes API Client Using A CertificateSigningRequest](https://kubernetes.io/docs/tasks/tls/certificate-issue-client-csr/)

Step 1: The question will give you csr and key so base64 encode it so you can use it in CertificateSigningRequest Object

```bash
# Command available in documentation
cat myuser.csr | base64 | tr -d "\n"
```

Step 2: Create the CSR Reqsource

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser # example
spec:
  request: <OUTPUT FROM STEP 1>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth
# kubectl apply -f john-csr.yaml
```

Approve the CSR Request:

```bash
kubectl certificate approve myuser
```

If question ask you to get the certifica exprted

```bash
# Command available in documentation
kubectl get csr myuser -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt
```

Finally you can create the role and rolebinding and use the --user=john in your binding
