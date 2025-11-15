# Kubernetes using kubeadm

## Installation Steps:

### Step 1

Always start with setting up the forwarding rules for **all the nodes**:

```json
https://kubernetes.io/docs/setup/production-environment/container-runtimes/#prerequisite-ipv4-forwarding-optional
```

How to reach in K8s Documentation:

```txt
Installing kubeadm --> Using kubeadm to Create a Cluster -->  container runtime
```

### Step 2:

Identify the Linux Distro we are using:

```bash
cat /etc/*release
or
cat /etc/os-release
```

### Step 3:

Installing kubeadm

```json
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
```

This steps need to be perform for both the nodes.

### Step 4:

Bootstrap a kubernetes cluster using kubeadm

- First we need to initialize the kubeadm
- We need interface IP to map for api-server

```bash
ip a
```

```bash
kubeadm init \
--apiserver-advertise-address=<eth0-IP-ADDRESS> \ --apiserver-cert-extra-sans=controlplane \
--pod-network-cidr=172.17.0.0/16  \
--service-cidr=172.20.0.0/16
```

### Step 5:

- kubeadm init will give you some commands that you need to run
- First command to set defautl kube config file

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Step 6

- Now our control plane is ready we need to join the worker nodes to the control plance node which can be done using the tokern provided by the kubneadm init command

```bash
kubeadm join 192.168.27.132:6443 --token l210r8.78om3j3gkee24d8u \
        --discovery-token-ca-cert-hash sha256:bcc8fc8685b0e5397bb66642a461e4fb94110a966fd69779fda820b5901cd8b4
```
