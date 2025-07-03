# 🧪 Kubernetes v1.32 Lab Setup for CKA Practice

Two Vagrant environments for setting up a Kubernetes v1.32 cluster using `kubeadm` on Ubuntu 22.04 VMs.

## Prerequisites

- **VirtualBox** (6.1+)
- **Vagrant** (2.2+)
- **6GB RAM** available
- **20GB free disk space**

## Setup Options

### Option 1: Pre-provisioned Setup

**File:** `Vagrantfile`

- Kubernetes tools pre-installed
- Ready for `kubeadm init`

### Option 2: Manual Setup

**File:** `Vagrantfile.manual`

- Clean VMs only
- Install everything manually

---

## 🚀 Pre-provisioned Setup

### 1. Start VMs

```bash
vagrant up
```

### 2. Initialize Master

```bash
vagrant ssh kmaster

# Initialize cluster
sudo kubeadm init --apiserver-advertise-address=192.168.56.10 --pod-network-cidr=10.244.0.0/16

# Configure kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install CNI
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Get join command
kubeadm token create --print-join-command
```

### 3. Join Workers

```bash
# SSH into each worker and run join command
vagrant ssh kworker1
sudo <join-command>
exit

vagrant ssh kworker2
sudo <join-command>
exit
```

### 4. Verify

```bash
kubectl get nodes
kubectl get pods -A
```

---

## 🛠️ Manual Setup

### 1. Start Clean VMs

```bash
vagrant up --vagrantfile=Vagrantfile.manual
```

### 2. Configure All Nodes (kmaster, kworker1, kworker2)

**Disable swap:**

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

**Load kernel modules:**

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

**Configure sysctl:**

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system
```

**Install containerd:**

```bash
sudo apt update
sudo apt install -y containerd

# Configure containerd
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

# Enable SystemdCgroup
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```

**Add Kubernetes repository:**

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg

# Add signing key
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

# Add repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

**Install Kubernetes tools:**

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
sudo systemctl enable kubelet
```

### 3. Initialize Master (kmaster only)

```bash
# Initialize cluster
sudo kubeadm init --apiserver-advertise-address=192.168.56.10 --pod-network-cidr=10.244.0.0/16

# Configure kubectl
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Install CNI
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# Get join command
kubeadm token create --print-join-command
```

### 4. Join Workers (kworker1, kworker2)

```bash
# Run join command from step 3
sudo kubeadm join 192.168.56.10:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
```

### 5. Verify Cluster

```bash
kubectl get nodes
kubectl get pods -A
kubectl cluster-info
```

---

## Network Configuration

| Node     | IP Address    |
| -------- | ------------- |
| kmaster  | 192.168.56.10 |
| kworker1 | 192.168.56.11 |
| kworker2 | 192.168.56.12 |

- **Pod Network:** 10.244.0.0/16
- **Service Network:** 10.96.0.0/12

---

## Common Commands

```bash
# Vagrant commands
vagrant up
vagrant ssh kmaster
vagrant status
vagrant halt
vagrant destroy

# Kubernetes commands
kubectl get nodes
kubectl get pods -A
kubectl cluster-info
kubeadm token create --print-join-command
```
