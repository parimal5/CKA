# Kubernetes Cluster Upgrade Guide

## Overview

This document provides step-by-step instructions for upgrading a Kubernetes cluster using kubeadm. The process involves upgrading the control plane nodes first, followed by worker nodes.

**Example:** Upgrading from v1.32.x to v1.33.x

## Prerequisites

- Administrative access to all cluster nodes
- Backup of your cluster (etcd snapshot recommended)
- Understanding of your cluster's current state and workloads

## Reference Documentation

- [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [pkgs.k8s.io: Kubernetes Community-Owned Package Repositories](https://v1-32.docs.kubernetes.io/blog/2023/08/15/pkgs-k8s-io-introduction/)

---

## Part 1: Control Plane Node Upgrade

### Step 1: Update Package Repository

Update the Kubernetes package repository to point to your target version:

```bash
# Replace 'v1.33' with your target version
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Add the GPG key for the new repository:

```bash
# Replace 'v1.33' with your target version
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### Step 2: Find Available Versions

Update package cache and check available kubeadm versions:

```bash
sudo apt update
sudo apt-cache madison kubeadm
```

### Step 3: Upgrade kubeadm

Install the target version of kubeadm:

```bash
# Replace '1.33.0-*' with your target version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.33.0-*' && \
sudo apt-mark hold kubeadm
```

Verify the installation:

```bash
kubeadm version
```

### Step 4: Verify Upgrade Plan

Check the upgrade plan to ensure everything looks correct:

```bash
sudo kubeadm upgrade plan
```

Review the output carefully for any warnings or issues.

### Step 5: Apply the Upgrade

Execute the cluster upgrade:

```bash
# Replace 'v1.33.0' with your target version
sudo kubeadm upgrade apply v1.33.0
```

> **Important:** This command upgrades the cluster control plane components including kube-apiserver, kube-controller-manager, kube-scheduler, and kube-proxy.

### Step 6: Drain the Control Plane Node

Prepare the node for kubelet upgrade by draining workloads:

```bash
# Replace 'controlplane' with your actual control plane node name
kubectl drain controlplane --ignore-daemonsets
```

### Step 7: Upgrade kubelet and kubectl

Install the new versions of kubelet and kubectl:

```bash
# Replace '1.33.0-*' with your target version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.33.0-*' kubectl='1.33.0-*' && \
sudo apt-mark hold kubelet kubectl
```

### Step 8: Restart kubelet

Reload systemd and restart the kubelet service:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

Verify kubelet is running:

```bash
sudo systemctl status kubelet
```

### Step 9: Uncordon the Control Plane Node

Allow workloads to be scheduled back to the control plane node:

```bash
# Replace 'controlplane' with your actual control plane node name
kubectl uncordon controlplane
```

Verify the node status:

```bash
kubectl get nodes
```

---

## Part 2: Worker Node Upgrade

Repeat the following steps for **each worker node** in your cluster.

### Step 1: Update Package Repository (On Worker Node)

SSH to the worker node and update the package repository:

```bash
# Replace 'v1.33' with your target version
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### Step 2: Upgrade kubeadm (On Worker Node)

```bash
# Replace '1.33.0-*' with your target version
sudo apt-mark unhold kubeadm && \
sudo apt-get update && sudo apt-get install -y kubeadm='1.33.0-*' && \
sudo apt-mark hold kubeadm
```

### Step 3: Drain the Worker Node (From Control Plane)

From the control plane node, drain the worker node:

```bash
# Replace 'worker-node-name' with the actual worker node name
kubectl drain worker-node-name --ignore-daemonsets
```

### Step 4: Upgrade Node Configuration (On Worker Node)

On the worker node, run the kubeadm upgrade:

```bash
sudo kubeadm upgrade node
```

### Step 5: Upgrade kubelet and kubectl (On Worker Node)

```bash
# Replace '1.33.0-*' with your target version
sudo apt-mark unhold kubelet kubectl && \
sudo apt-get update && sudo apt-get install -y kubelet='1.33.0-*' kubectl='1.33.0-*' && \
sudo apt-mark hold kubelet kubectl
```

### Step 6: Restart kubelet (On Worker Node)

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

### Step 7: Uncordon the Worker Node (From Control Plane)

Allow workloads to be scheduled back to the worker node:

```bash
# Replace 'worker-node-name' with the actual worker node name
kubectl uncordon worker-node-name
```

---

## Post-Upgrade Verification

### Verify Cluster Status

Check that all nodes are ready and running the new version:

```bash
kubectl get nodes -o wide
```

### Verify Component Versions

```bash
kubectl version --short
kubeadm version
kubelet --version
```

### Check Cluster Health

```bash
kubectl get pods -A
kubectl cluster-info
```

---

## Key Points and Best Practices

### Before You Start

- **Backup your cluster:** Always create an etcd snapshot before upgrading
- **Read release notes:** Review Kubernetes release notes for breaking changes
- **Test in staging:** Perform the upgrade in a non-production environment first
- **Plan maintenance window:** Schedule upgrades during low-traffic periods

### During Upgrade

- **One minor version at a time:** Don't skip minor versions (e.g., 1.31 → 1.32 → 1.33)
- **Control plane first:** Always upgrade control plane nodes before worker nodes
- **One node at a time:** Upgrade worker nodes sequentially to maintain availability
- **Monitor carefully:** Watch for any errors or warnings during each step

### Version Considerations

- Replace all instances of `v1.33` and `1.33.0` with your target version
- Ensure your target version is supported and compatible with your workloads
- Check addon compatibility (CNI, CSI drivers, monitoring tools, etc.)

### Troubleshooting

- If a step fails, investigate the error before proceeding
- Check kubelet logs: `sudo journalctl -u kubelet -f`
- Verify network connectivity and DNS resolution
- Ensure sufficient resources (CPU, memory, disk space) on all nodes

### Rollback Considerations

- Keep the previous kubeadm version available for potential rollback
- Document your current configuration before starting
- Have a tested rollback procedure ready

---

## Additional Resources

- [Kubernetes Version Skew Policy](https://kubernetes.io/releases/version-skew-policy/)
- [Upgrading Linux nodes / Worker nodes](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/upgrading-linux-nodes/)
- [Backing up an etcd cluster](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
