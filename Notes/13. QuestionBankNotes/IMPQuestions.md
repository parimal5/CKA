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
