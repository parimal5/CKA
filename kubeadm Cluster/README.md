# 🧪 Kubernetes Lab with Vagrant & VirtualBox

This project provides two Vagrant-based environments for setting up a local multi-node Kubernetes cluster using `kubeadm` on Ubuntu 22.04 virtual machines.

---

## 📁 Project Structure

```bash
.
├── Vagrantfile # Fully provisioned setup (auto install K8s tools)
├── Vagrantfile.manual # Manual setup (clean OS only)
└── README.md
```

## 🚀 Option 1: Provisioned Setup [Vagrantfile](./Vagrantfile)

This setup:

- Spins up 3 Ubuntu 22.04 VMs: `kmaster`, `kworker1`, `kworker2`
- Automatically:
  - Disables swap
  - Installs `containerd`
  - Installs `kubeadm`, `kubelet`, `kubectl`
  - Prepares each node for Kubernetes

### Usage:

```bash
vagrant up
```

Once VMs are ready:

1. SSH into master:

```bash
vagrant ssh kmaster
```

2. Run kubeadm init with networking:

```bash
sudo kubeadm init --apiserver-advertise-address=192.168.56.10 --pod-network-cidr=192.168.0.0/16
```

3. Set up kubectl config on master.
4. Copy the kubeadm join command.
5. SSH into each worker node and join the cluster.
6. Install CNI (e.g., Calico) from master.

## 🛠️ Option 2: Manual Setup [Vagrantfile.manual](./Vagrantfile.manual)

This setup:

- Spins up the same 3 VMs
- Does NOT install anything
- You must configure everything manually (ideal for CKA practice)

Usage:

```bash
vagrant up --provider=virtualbox --vagrantfile=Vagrantfile.manual
```

Then SSH into each VM:

```bash
vagrant ssh kmaster
vagrant ssh kworker1
vagrant ssh kworker2
```

---

## 🛠️ Manual Kubernetes Setup Checklist (For Vagrantfile.manual)

Follow these steps manually on the VMs after booting them via `vagrant up`:

### 🔹 On All Nodes (Master & Workers)

- Disable swap
- Remove swap entry from `/etc/fstab`
- Load kernel modules: `overlay`, `br_netfilter`
- Set sysctl parameters for Kubernetes networking
- Install and configure container runtime (e.g., containerd)
- Enable and start container runtime service
- Add Kubernetes APT repository
- Install Kubernetes tools: `kubelet`, `kubeadm`, `kubectl`
- Hold package versions to prevent unintended upgrades

### 🔹 On Master Node Only

- Initialize the cluster using `kubeadm init`
- Configure kubectl access using `admin.conf`
- Install a CNI plugin (e.g., Calico or Flannel)

### 🔹 On Worker Nodes Only

- Join the cluster using the `kubeadm join` command output from the master

### 🔹 Final Steps (On Master)

- Check cluster status with `kubectl get nodes`
- Verify pod network with `kubectl get pods -A`
- Optionally deploy a test application to validate setup
