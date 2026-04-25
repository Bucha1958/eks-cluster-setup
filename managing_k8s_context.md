# Managing Multiple Kubernetes Contexts (k3s + EKS) on a Single EC2 Instance

## Overview

This document explains how to manage Kubernetes context conflicts when running both a local k3s cluster and an AWS EKS cluster on the same EC2 instance.

**Problem**

After installing and configuring an EKS cluster using eksctl, kubectl did not automatically point to the EKS cluster.

Instead, it defaulted to the existing k3s installation, causing errors such as:

***error:*** error loading config file "/etc/rancher/k3s/k3s.yaml": permission denied

**Root Cause**

k3s installs its kubeconfig at:

```bash
/etc/rancher/k3s/k3s.yaml
```

**EKS uses a separate kubeconfig:**

```bash
~/.kube/config
```


kubectl defaults to whichever kubeconfig is detected first
No KUBECONFIG environment variable was set
Solution
### 1. Generate EKS kubeconfig

aws eks update-kubeconfig \
  --region us-east-1 \
  --name dev-cluster
### 2. Check available contexts
kubectl config get-contexts
### 3. Switch to EKS context
kubectl config use-context arn:aws:eks:us-east-1:...:cluster/dev-cluster
### 4. Verify cluster
kubectl get nodes

Expected output shows EKS worker nodes in Ready state.

Permanent Fix
**Option 1 (Recommended)**

Add to ~/.bashrc:
```bash
export KUBECONFIG=$HOME/.kube/config
```

Then reload:
```bash
source ~/.bashrc
```

**Option 2 (Multi-cluster setup)**

```bash
export KUBECONFIG=$HOME/.kube/config:/etc/rancher/k3s/k3s.yaml
```

Switch contexts using:
```bash
kubectl config use-context <context-name>
```

**Key Learning**

* ***Multiple Kubernetes installations on the same host can cause kubeconfig conflicts***
* ***kubectl uses the first available kubeconfig unless explicitly configured***
* ***AWS EKS requires explicit context switching via kubeconfig***
* ***Proper kubeconfig management is critical in multi-cluster environments***

**Outcome**

***EKS cluster successfully accessed***
***k3s cluster still available locally***
***kubectl context properly switched between environments***