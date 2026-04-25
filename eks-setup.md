#  EKS Production Simulation (DevOps Project)

##  Project Overview

This project demonstrates the end-to-end creation and troubleshooting of an AWS EKS (Elastic Kubernetes Service) cluster using `eksctl`.

The focus is not just on provisioning the cluster, but on **debugging real-world infrastructure and configuration issues**, similar to what is encountered in production environments.

---

##  Architecture

* **EKS Cluster** (Managed Kubernetes Control Plane)
* **Managed Node Group** (EC2 instances)
* **Auto-created VPC** (via eksctl)
* **Kubernetes Add-ons**

  * CoreDNS
  * kube-proxy
  * VPC CNI
  * metrics-server

---

##  Tools & Technologies

* AWS CLI
* eksctl
* kubectl
* EC2 (t3.small instances)
* CloudFormation (used internally by EKS)

---

##  Setup Steps

### 1. Install AWS CLI

Initial installation via apt failed:

```bash
sudo apt install awscli -y
```

**Fix:** Installed using official AWS binary:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

---

### 2. Configure AWS Credentials

```bash
aws configure
```

Verified:

```bash
aws sts get-caller-identity
```

---

### 3. Create EKS Cluster

```bash
eksctl create cluster \
  --name dev-cluster \
  --version 1.31 \
  --region us-east-1 \
  --nodegroup-name workers \
  --node-type t3.small \
  --nodes 2 \
  --zones us-east-1a,us-east-1b
```

---

### 4. Configure kubectl Access

```bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name dev-cluster
```

---

### 5. Verify Cluster

```bash
kubectl get nodes
```

Output:

```
ip-192-168-18-74   Ready
ip-192-168-52-31   Ready
```

---

##  Issues Encountered & Fixes

###  Issue 1: awscli not available via apt

**Error:**

```
Package 'awscli' has no installation candidate
```

**Cause:**

* Package not available in Ubuntu repo

**Fix:**

* Installed via official AWS CLI binary

---

### Issue 2: EKS Cluster Creation Failed

**Error:**

```
exceeded max wait time for StackCreateComplete waiter
```

**Observation:**

* Control plane created successfully
* Nodegroup stack failed and rolled back

**Root Cause:**

* EC2 provisioning failure using `t3.medium`
* Likely due to:

  * instance capacity issue
  * or account-level resource limits

**Fix:**

* Changed instance type to `t3.small`

**Result:**

* Nodegroup created successfully
* Cluster became fully functional

---

### Issue 3: kubectl pointing to K3s instead of EKS

**Error:**

```
Unable to read /etc/rancher/k3s/k3s.yaml
```

**Cause:**

* Existing K3s installation overriding kubeconfig
* kubectl using wrong config file

**Fix:**

```bash
export KUBECONFIG=$HOME/.kube/config
kubectl config use-context arn:aws:eks:us-east-1:<ACCOUNT_ID>:cluster/dev-cluster
```

**Result:**

* kubectl successfully connected to EKS cluster

---

### Issue 4: Using sudo with kubectl

**Problem:**

* `sudo kubectl` used root kubeconfig
* Different from user kubeconfig

**Fix:**

* Avoid using `sudo kubectl`
* Use standard user context

---

## Key Learnings

* EKS uses **CloudFormation under the hood**
* Nodegroup failures are often **EC2-related**, not Kubernetes-related
* Instance type selection can impact provisioning success
* kubeconfig misconfiguration can lead to connecting to the wrong cluster
* Avoid mixing multiple Kubernetes environments (K3s vs EKS) without managing contexts

---

## Real-World Insight

This project simulates real DevOps scenarios:

* Infrastructure provisioning failures
* Debugging AWS resource constraints
* Resolving cluster connectivity issues
* Managing multiple Kubernetes environments

---

## Next Steps

* Deploy applications to EKS
* Expose services using LoadBalancer
* Configure Ingress
* Integrate CI/CD pipeline
* Monitor cluster using Prometheus/Grafana

---

## 👨‍💻 Author

DevOps Engineer focused on cloud infrastructure, Kubernetes, and production troubleshooting.
