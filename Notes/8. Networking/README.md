<div align="center">
  <h1><strong>CKA Exam Notes</strong></h1>
  <h3>Networking</h3>
</div>

## Basic

![alt text](image.png)

Q. How to check if the node is conected to which Network Interface?

Note the Internal IP address if the node:

```bash
k get nodes - o wide
```

**Output**:

```bash
NAME           STATUS   ROLES           AGE   VERSION   INTERNAL-IP      EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
controlplane   Ready    control-plane   34m   v1.33.0   192.168.63.147   <none>        Ubuntu 22.04.5 LTS   5.15.0-1083-gcp   containerd://1.6.26
node01         Ready    <none>          33m   v1.33.0   192.168.139.13   <none>        Ubuntu 22.04.5 LTS   5.15.0-1083-gcp   containerd://1.6.26
```

Check which network interface has that IP included.

```bash
ip address
```

**Output**:

````bash
3: eth0@if2063: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1410 qdisc noqueue state UP group default
    link/ether be:b9:ea:ae:ce:84 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.63.147/32 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::bcb9:eaff:feae:ce84/64 scope link
       valid_lft forever preferred_lft forever
       ```
````

How to check the any type of interface(eg. bridge) on you machine?

```bash
ip address show type bridge
```

How to check the port of services on your machine?

```bash
netstat -h # Help

netstat -tnlp
```
