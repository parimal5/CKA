# CKA Exam - Complete Study Plan & Practice Guide

## 📋 Exam Overview (2025 Format)

**Key Facts:**
- 17 questions, 2 hours duration, 66% pass mark
- Hands-on performance-based exam (no multiple choice)
- Linux Foundation provides simulator with 17 questions per attempt
- One free retake included
- Updated after November 25, 2024 with new topics like Gateway API, Helm, Kustomize, CRDs & Operators

**Exam Domains & Weightings:**
The core domains are Storage, Troubleshooting, Workloads & Scheduling, Cluster Architecture, Services & Networking

---

## 🎯 4-Week Study Plan

### Week 1: Foundation & Cluster Setup
**Focus Areas:**
- Cluster Architecture, Installation & Configuration (25%)
- Basic kubectl commands and cluster management

**Daily Practice (2-3 hours):**

**Day 1-2: Cluster Basics**
- Install kubeadm, kubelet, kubectl
- Initialize a cluster with kubeadm
- Add worker nodes
- Configure kubectl contexts

**Timed Challenge (15 mins):**
```bash
# Challenge: Set up a 2-node cluster
1. Install Kubernetes components on master and worker
2. Initialize cluster with pod network CIDR 10.244.0.0/16
3. Install Flannel network plugin
4. Join worker node
5. Verify all nodes are Ready
```

**Day 3-4: etcd & Cluster Management**
- Backup and restore etcd
- Upgrade cluster components
- Manage cluster certificates

**Timed Challenge (10 mins):**
```bash
# Challenge: etcd Backup & Restore
1. Create etcd backup
2. Create a test deployment
3. Restore etcd from backup
4. Verify deployment is gone after restore
```

**Day 5-7: Node Management**
- Drain and cordon nodes
- Node maintenance
- Static pods
- Kubelet configuration

---

### Week 2: Workloads & Scheduling
**Focus Areas:**
- Workloads & Scheduling (15%)
- Deployments, DaemonSets, Jobs, CronJobs

**Daily Practice:**

**Day 1-2: Pod Management**
- Pod creation with various specs
- Pod lifecycle and troubleshooting
- Init containers and sidecar patterns

**Timed Challenge (8 mins):**
```bash
# Challenge: Multi-container Pod
1. Create pod with nginx + busybox sidecar
2. Sidecar should run: while true; do date >> /shared/log; sleep 5; done
3. Both containers share /shared volume
4. Verify log file is being written
```

**Day 3-4: Deployments & Scaling**
- Deployment strategies
- Rolling updates and rollbacks
- HPA and VPA

**Timed Challenge (10 mins):**
```bash
# Challenge: Deployment Management
1. Create deployment 'web' with nginx:1.19, 3 replicas
2. Expose via NodePort service on port 80
3. Update image to nginx:1.20
4. Scale to 5 replicas
5. Rollback to previous version
6. Check rollout history
```

**Day 5-7: Jobs & DaemonSets**
- Batch processing with Jobs
- CronJobs for scheduled tasks
- DaemonSets for node-level services

---

### Week 3: Services, Networking & Storage
**Focus Areas:**
- Services & Networking (20%)
- Storage (10%)

**Daily Practice:**

**Day 1-2: Services & Load Balancing**
- ClusterIP, NodePort, LoadBalancer
- Service discovery
- Endpoint management

**Timed Challenge (12 mins):**
```bash
# Challenge: Service Networking
1. Create 2 deployments: frontend (nginx) and backend (httpd)
2. Create ClusterIP service for backend on port 8080
3. Create NodePort service for frontend on port 80
4. Configure frontend to proxy requests to backend
5. Test connectivity from frontend pod to backend service
```

**Day 3-4: Network Policies**
- Ingress and egress rules
- Pod-to-pod communication control
- Namespace isolation

**Day 5-7: Storage Management**
- PVs and PVCs
- Storage classes
- Volume types and mounting

**Timed Challenge (15 mins):**
```bash
# Challenge: Persistent Storage
1. Create PV with hostPath /data, 2Gi capacity
2. Create PVC requesting 1Gi storage
3. Create pod using PVC mounted at /app/data
4. Write test file to verify persistence
5. Delete pod and recreate - verify file exists
```

---

### Week 4: Security, Monitoring & Troubleshooting
**Focus Areas:**
- Troubleshooting (30%)
- Cluster Maintenance & Security

**Daily Practice:**

**Day 1-2: RBAC & Security**
- Users, groups, and service accounts
- Roles and ClusterRoles
- Security contexts

**Timed Challenge (10 mins):**
```bash
# Challenge: RBAC Setup
1. Create service account 'developer'
2. Create role allowing get, list, create on pods
3. Bind role to service account
4. Create pod using this service account
5. Test permissions with kubectl auth can-i
```

**Day 3-4: Monitoring & Logging**
- Resource monitoring
- Log aggregation
- Metrics server

**Day 5-7: Troubleshooting**
- Pod troubleshooting
- Network debugging
- Node and cluster issues

**Timed Challenge (20 mins):**
```bash
# Challenge: Multi-level Troubleshooting
1. Fix broken pod (wrong image, resource limits)
2. Debug service connectivity issue
3. Resolve node NotReady state
4. Fix RBAC permission denied error
5. Restore failed deployment rollout
```

---

## ⚡ Speed Optimization Techniques

### Essential Aliases & Shortcuts
```bash
# Add to ~/.bashrc
alias k=kubectl
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgn='kubectl get nodes'
alias kd='kubectl describe'
alias kdel='kubectl delete'
export do="--dry-run=client -o yaml"
export force="--force --grace-period=0"
```

### Imperative Command Mastery
```bash
# Pods
kubectl run nginx --image=nginx --port=80 --labels=app=web

# Deployments  
kubectl create deployment web --image=nginx --replicas=3

# Services
kubectl expose pod nginx --port=80 --type=NodePort

# Jobs
kubectl create job test --image=busybox -- /bin/sh -c 'echo hello'

# CronJobs
kubectl create cronjob backup --image=busybox --schedule="0 2 * * *" -- /bin/sh -c 'echo backup'

# Generate YAML for editing
kubectl run nginx --image=nginx $do > pod.yaml
kubectl create deployment web --image=nginx $do > deploy.yaml
```

### Documentation Navigation
**Bookmark these sections:**
- Pod spec: `kubernetes.io/docs/reference/generated/kubernetes-api/v1.28/#pod-v1-core`
- Service spec: `kubernetes.io/docs/concepts/services-networking/service/`
- NetworkPolicy: `kubernetes.io/docs/concepts/services-networking/network-policies/`
- RBAC: `kubernetes.io/docs/reference/access-authn-authz/rbac/`

**Quick kubectl explain:**
```bash
kubectl explain pod.spec.containers
kubectl explain deployment.spec.strategy
kubectl explain service.spec
```

---

## 📝 Practice Test Schedule

### Week 1-2: Foundation Building
- Daily: 3-5 basic challenges (30-45 mins)
- Weekend: Complete domain-specific test (2 hours)

### Week 3: Integration Practice  
- Daily: Mixed challenges covering multiple domains (45-60 mins)
- Mid-week: Full mock exam (2 hours)

### Week 4: Exam Simulation
- Daily: Timed scenario-based challenges (60 mins)
- 3 full mock exams with exam conditions
- Review and weakness identification

### Final Week Before Exam
- 1 mock exam every other day
- Focus on weak areas identified
- Speed optimization drills
- Documentation navigation practice

---

## 🎯 Sample Exam-Level Challenges

### Challenge Set 1: Basic Operations (45 mins)
1. **Cluster Setup** (8 mins): Add new worker node to existing cluster
2. **Pod Management** (5 mins): Create multi-container pod with shared volume
3. **Service Creation** (7 mins): Expose deployment with specific requirements
4. **Storage** (10 mins): Create PVC and mount in pod
5. **RBAC** (8 mins): Create service account with limited permissions
6. **Troubleshooting** (7 mins): Fix broken deployment

### Challenge Set 2: Advanced Scenarios (60 mins)
1. **Network Policy** (12 mins): Implement pod-to-pod communication rules
2. **etcd Backup** (8 mins): Backup and restore cluster state
3. **Rolling Update** (10 mins): Update deployment with zero downtime
4. **Node Maintenance** (8 mins): Safely drain node for maintenance
5. **Job Processing** (10 mins): Create batch job with specific requirements
6. **Complex Troubleshooting** (12 mins): Multi-component failure scenario

---

## 🔧 Environment Setup

### Local Practice Environment
**Option 1: kubeadm cluster (Recommended)**
- 3 VMs (1 master, 2 workers)
- Ubuntu 20.04+ or CentOS 8+
- 2GB RAM, 2 CPUs minimum per VM

**Option 2: Managed Solutions**
- Kind (Kubernetes in Docker)
- K3s for lightweight setup
- Minikube for single-node

### Resource Requirements
- Practice environment matching exam conditions
- Bookmark management for quick doc access
- Timer for tracking challenge completion
- Note-taking system for command patterns

---

## 📚 Essential Commands Cheat Sheet

### Context & Cluster
```bash
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl cluster-info
kubectl get nodes -o wide
```

### Pod Operations
```bash
kubectl get pods -A -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name> -c <container>
kubectl exec -it <pod-name> -- /bin/bash
kubectl port-forward <pod-name> 8080:80
```

### Troubleshooting
```bash
kubectl get events --sort-by=.metadata.creationTimestamp
kubectl describe node <node-name>
kubectl top pods --sort-by=memory
kubectl logs <pod-name> --previous
```

### Resource Management
```bash
kubectl scale deployment <name> --replicas=5
kubectl rollout status deployment/<name>
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
```

---

## ✅ Pre-Exam Checklist

**1 Week Before:**
- [ ] Complete 3 full mock exams scoring 75%+
- [ ] Identify and practice weak areas
- [ ] Verify all kubectl shortcuts work
- [ ] Practice documentation navigation

**1 Day Before:**
- [ ] Test exam environment setup
- [ ] Review imperative commands list
- [ ] Practice context switching
- [ ] Get good rest!

**Exam Day:**
- [ ] Start with easy questions to build confidence
- [ ] Skip difficult questions, return later
- [ ] Use imperative commands when possible
- [ ] Verify solutions before moving on
- [ ] Save 15 minutes for final review

---

## 🎓 Success Tips

1. **Time Management**: Allocate ~7 minutes per question average
2. **Command Efficiency**: Master imperative commands for speed
3. **Documentation**: Practice finding answers quickly in k8s docs
4. **Verification**: Always verify your solutions work
5. **Context Switching**: Practice switching between cluster contexts
6. **Troubleshooting**: Develop systematic debugging approach
7. **Stay Calm**: Skip hard questions and return to them

Remember: The CKA is about demonstrating real-world Kubernetes administration skills. Focus on hands-on practice over theory, and always verify your solutions work in practice!
