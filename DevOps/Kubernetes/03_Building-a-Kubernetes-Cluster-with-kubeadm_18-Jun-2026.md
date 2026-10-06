# Kubernetes Class 03 - Building a Kubernetes Cluster with kubeadm

> **Class Date:** 18-Jun-2026
>
> **Module:** Kubernetes Cluster Setup using kubeadm

---

# 📚 Table of Contents

1. Introduction
2. Why Learn kubeadm?
3. Kubernetes Cluster Architecture
4. Lab Environment
5. Prerequisites
6. Preparing Ubuntu Servers
7. Hostname Configuration
8. Configure `/etc/hosts`
9. Disable Swap
10. Load Required Kernel Modules
11. Configure Kernel Parameters
12. Verify System Configuration
13. Summary

---

# 1. Introduction

In the previous classes, we learned:

- What Kubernetes is
- Why Kubernetes was created
- Kubernetes Architecture
- CRI
- CNI
- CSI
- Installation Methods

Now it's time to build our own Kubernetes Cluster.

This chapter explains how to install a **production-like Kubernetes cluster** using **kubeadm**.

> **Goal**
>
> Build a Kubernetes cluster from scratch and understand every command used during the installation.

---

# 2. Why Learn kubeadm?

There are many ways to create a Kubernetes cluster.

- Minikube
- Kind
- AKS
- EKS
- GKE
- K3s
- OpenShift

So why learn **kubeadm**?

Because kubeadm helps you understand how Kubernetes works internally.

---

## What is kubeadm?

**kubeadm** is an official Kubernetes tool used to bootstrap a Kubernetes cluster.

It automates many complex tasks such as:

- Initializing the Control Plane
- Generating Certificates
- Creating kubeconfig files
- Starting Control Plane Components
- Generating Worker Join Tokens

---

## Real World Example

Imagine building a house.

You don't buy a fully furnished apartment if you want to understand construction.

Instead, you build the house brick by brick.

kubeadm works the same way.

Managed Kubernetes services hide everything.

kubeadm teaches you everything.

---

## Why Companies Use kubeadm

Many organizations still deploy self-managed Kubernetes clusters using kubeadm because they need:

- Full Control
- Private Data Centers
- Air-Gapped Environments
- Government Projects
- Enterprise Security
- Cost Optimization

---

# 3. Kubernetes Cluster Architecture

Our cluster will look like this.

```text
                    Kubernetes Cluster

              +-------------------------+
              |      Control Plane      |
              |-------------------------|
              | API Server              |
              | Scheduler               |
              | Controller Manager      |
              | etcd                    |
              +-------------------------+

                    /             \

                   /               \

          +---------------+   +---------------+
          | Worker Node 1 |   | Worker Node 2 |
          +---------------+   +---------------+
          | kubelet       |   | kubelet       |
          | kube-proxy    |   | kube-proxy    |
          | containerd    |   | containerd    |
          +---------------+   +---------------+

                  Pods              Pods
```

---

# 4. Lab Environment

We will use the following lab setup.

| Component | Value |
|------------|-------|
| OS | Ubuntu 24.04 LTS |
| Control Plane | 1 |
| Worker Nodes | 2 |
| CPU | 2 vCPU |
| RAM | 4 GB |
| Storage | 40 GB |
| Network | Private Network |

Cloud Providers:

- Azure
- AWS
- GCP
- VMware
- VirtualBox

---

# 5. Prerequisites

Before installing Kubernetes, ensure the following requirements are met.

## Operating System

Supported Linux distributions include:

- Ubuntu
- RHEL
- Rocky Linux
- AlmaLinux
- Debian

Ubuntu LTS is recommended for learning.

---

## Hardware Requirements

### Control Plane

- 2 CPU
- 4 GB RAM
- 40 GB Storage

### Worker Node

- 2 CPU
- 2–4 GB RAM
- 20 GB Storage

---

## Network Requirements

All nodes must communicate with each other.

Ensure:

- Internet Connectivity
- DNS Resolution
- Required Ports Open
- Static IP Addresses (Recommended)

---

# 6. Preparing Ubuntu Servers

Always update your operating system before installing Kubernetes.

```bash
sudo apt update
```

Update installed packages.

```bash
sudo apt upgrade -y
```

Remove unused packages.

```bash
sudo apt autoremove -y
```

Reboot if required.

```bash
sudo reboot
```

---

## Why Update First?

Updating the system ensures:

- Latest Security Patches
- Bug Fixes
- Package Compatibility
- Stable Installation

Skipping updates may cause dependency issues later.

---

# 7. Configure Hostnames

Each node should have a meaningful hostname.

Example:

| Machine | Hostname |
|----------|----------|
| Master | master |
| Worker 1 | worker1 |
| Worker 2 | worker2 |

Set hostname.

```bash
sudo hostnamectl set-hostname master
```

Worker example.

```bash
sudo hostnamectl set-hostname worker1
```

Verify.

```bash
hostname
```

Expected Output

```text
master
```

---

# 8. Configure /etc/hosts

Every node should know the IP addresses of the other nodes.

Edit:

```bash
sudo nano /etc/hosts
```

Example.

```text
10.0.0.10 master

10.0.0.11 worker1

10.0.0.12 worker2
```

Verify.

```bash
ping master

ping worker1

ping worker2
```

---

## Why is /etc/hosts Important?

Without proper hostname resolution:

- Nodes may fail to communicate.
- Worker nodes may fail to join the cluster.
- kubeadm may report connectivity issues.

---

# 9. Disable Swap

One of the most important prerequisites.

Check swap.

```bash
free -h
```

Disable swap immediately.

```bash
sudo swapoff -a
```

Disable permanently.

```bash
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

Verify.

```bash
free -h
```

Expected Output

```text
Swap: 0B
```

---

## Why Does Kubernetes Require Swap to be Disabled?

Kubernetes relies on predictable memory management.

If swap is enabled:

- Performance becomes unpredictable.
- kubelet may refuse to start.
- Resource scheduling becomes inaccurate.

---

# 10. Load Required Kernel Modules

Kubernetes networking requires specific Linux kernel modules.

Create the configuration file.

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

Load the modules.

```bash
sudo modprobe overlay

sudo modprobe br_netfilter
```

Verify.

```bash
lsmod | grep overlay

lsmod | grep br_netfilter
```

---

## What is the overlay Module?

The **overlay** kernel module enables OverlayFS, which is used by container runtimes (such as containerd) to efficiently manage container image layers.

Without it, containers cannot efficiently share image layers.

---

## What is br_netfilter?

The **br_netfilter** module allows bridged network traffic to pass through Linux firewall rules (`iptables`), enabling Kubernetes networking and network policies to function correctly.

---

# 11. Configure Kernel Parameters (sysctl)

Create the Kubernetes sysctl configuration.

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF
```

Apply the configuration.

```bash
sudo sysctl --system
```

Verify.

```bash
sysctl net.ipv4.ip_forward
```

Expected Output

```text
net.ipv4.ip_forward = 1
```

---

## Why are These Parameters Required?

### net.ipv4.ip_forward

Allows packets to be forwarded between network interfaces, which is essential for Pod-to-Pod communication across nodes.

### bridge-nf-call-iptables

Ensures that bridged network traffic is processed by `iptables`, allowing Kubernetes Services and Network Policies to work as expected.

---

# 12. Verify System Configuration

Run these checks before proceeding:

```bash
hostname
free -h
lsmod | grep overlay
lsmod | grep br_netfilter
sysctl net.ipv4.ip_forward
```

Checklist:

- ✅ Hostname configured
- ✅ `/etc/hosts` updated
- ✅ Swap disabled
- ✅ Required kernel modules loaded
- ✅ sysctl settings applied
- ✅ System packages updated

---

# Chapter Summary (Part 1)

In this part, we learned:

- Why kubeadm is important
- Kubernetes cluster architecture
- Lab requirements
- Preparing Ubuntu servers
- Configuring hostnames
- Updating `/etc/hosts`
- Disabling swap
- Loading required kernel modules
- Configuring sysctl parameters
- Verifying system readiness

The system is now fully prepared for installing the container runtime and Kubernetes components.

---

# 13. Install Container Runtime

## What is a Container Runtime?

A Container Runtime is software responsible for running containers.

Kubernetes does **not** run containers directly.

Instead, Kubernetes communicates with a Container Runtime through the **Container Runtime Interface (CRI)**.

```
                Kubernetes

                     │

                     ▼

          Container Runtime Interface

                     │

                     ▼

                containerd

                     │

                     ▼

              Linux Kernel
```

Popular Container Runtimes:

| Runtime | Status |
|----------|---------|
| containerd | ⭐ Recommended |
| CRI-O | Popular |
| Docker Engine | Requires cri-dockerd |
| Podman | Supported in some environments |

---

## Why containerd?

Earlier versions of Kubernetes used Docker as the default runtime.

Starting from Kubernetes v1.24:

- Dockershim was removed.
- containerd became the preferred runtime.
- It is lightweight.
- Faster startup.
- Better Kubernetes integration.

---

## Install containerd

Update package index.

```bash
sudo apt update
```

Install containerd.

```bash
sudo apt install -y containerd
```

Verify installation.

```bash
containerd --version
```

Example Output

```text
containerd github.com/containerd/containerd 2.x.x
```

---

## Generate Default Configuration

Create configuration directory.

```bash
sudo mkdir -p /etc/containerd
```

Generate configuration.

```bash
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

---

## Configure Systemd Cgroup

Open configuration.

```bash
sudo nano /etc/containerd/config.toml
```

Find

```text
SystemdCgroup = false
```

Change to

```text
SystemdCgroup = true
```

---

## Why SystemdCgroup?

Kubernetes uses systemd to manage services.

Using the same cgroup driver avoids:

- Memory Issues
- CPU Scheduling Issues
- kubelet Errors

---

## Restart containerd

```bash
sudo systemctl restart containerd

sudo systemctl enable containerd
```

Verify.

```bash
systemctl status containerd
```

Expected

```text
active (running)
```

---

# 14. Install Kubernetes Components

Three important packages are required.

| Package | Purpose |
|----------|----------|
| kubeadm | Creates Cluster |
| kubelet | Runs Pods |
| kubectl | Command Line Tool |

---

## Install Packages

```bash
sudo apt install -y kubelet kubeadm kubectl
```

---

## Prevent Automatic Upgrade

```bash
sudo apt-mark hold kubeadm kubelet kubectl
```

---

## Why Hold Packages?

Imagine

Master Node

Version

```
1.34
```

Worker Node automatically upgrades

```
1.35
```

Now

Cluster Version Mismatch

↓

Unexpected Issues

Holding packages prevents accidental upgrades.

---

## Verify Installation

```bash
kubeadm version

kubectl version --client

kubelet --version
```

---

# 15. Initialize Kubernetes Control Plane

This command is executed **ONLY** on the Master Node.

```bash
sudo kubeadm init \
--pod-network-cidr=10.244.0.0/16
```

---

## What Happens Internally?

kubeadm performs many tasks.

1. Pre-flight Checks

2. Generate Certificates

3. Generate kubeconfig

4. Create etcd

5. Start API Server

6. Start Scheduler

7. Start Controller Manager

8. Generate Join Token

9. Display Join Command

---

## Architecture After Initialization

```
           Master Node

API Server

Scheduler

Controller Manager

etcd

↓

Waiting for Worker Nodes
```

---

## Important Output

Near the end you'll see

```bash
kubeadm join 10.0.0.10:6443 \
--token abc.xyz \
--discovery-token-ca-cert-hash sha256:xxxxxxxx
```

⚠️ Save this command.

It will be required on every Worker Node.

---

# 16. Configure kubectl

By default kubectl cannot communicate with the cluster.

Configure it.

Create directory.

```bash
mkdir -p $HOME/.kube
```

Copy configuration.

```bash
sudo cp /etc/kubernetes/admin.conf \
$HOME/.kube/config
```

Change ownership.

```bash
sudo chown $(id -u):$(id -g) \
$HOME/.kube/config
```

---

## Verify

```bash
kubectl cluster-info
```

Example

```
Kubernetes control plane is running
```

---

## Check Nodes

```bash
kubectl get nodes
```

Initially

```
STATUS

NotReady
```

Don't worry.

This is expected.

Reason:

CNI Plugin not installed.

---

# 17. Install CNI Plugin

Pods need networking.

Without networking

Pods cannot communicate.

Most beginners forget this step.

---

## What is CNI?

Container Network Interface

Responsibilities

- Pod IP
- Routing
- Pod Communication
- Network Policies

---

## Flannel

Flannel is simple.

Perfect for beginners.

Install Flannel.

```bash
kubectl apply -f \
https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

---

## Verify

```bash
kubectl get pods -n kube-flannel
```

Expected

```
Running
```

---

## Check Nodes Again

```bash
kubectl get nodes
```

Now

```
STATUS

Ready
```

Congratulations!

Your Control Plane is operational.

---

# 18. Join Worker Nodes

Run the join command generated earlier.

Example

```bash
sudo kubeadm join 10.0.0.10:6443 \
--token abc.xyz \
--discovery-token-ca-cert-hash sha256:xxxxxxxx
```

Run this on

- worker1

- worker2

---

## What Happens During Join?

Worker contacts

↓

API Server

↓

Certificate Validation

↓

Node Registration

↓

kubelet Starts

↓

Node becomes Ready

---

## Verify Cluster

On Master

```bash
kubectl get nodes
```

Expected

```
NAME

master

worker1

worker2

STATUS

Ready

Ready

Ready
```

---

# 19. Verify Cluster Health

Useful commands

```bash
kubectl get nodes
```

```bash
kubectl get pods -A
```

```bash
kubectl cluster-info
```

```bash
kubectl get namespaces
```

```bash
kubectl get componentstatuses
```

---

## Verify System Pods

```bash
kubectl get pods -n kube-system
```

You should see

- CoreDNS

- kube-proxy

- etcd

- kube-apiserver

- kube-controller-manager

- kube-scheduler

All should be

```
Running
```

---

# 20. Common Installation Errors

## Swap Enabled

Error

```
Running with swap on is not supported.
```

Solution

```bash
sudo swapoff -a
```

---

## CNI Not Installed

Symptoms

```
Node NotReady
```

Solution

Install Flannel (or another CNI plugin).

---

## containerd Not Running

Check

```bash
systemctl status containerd
```

Restart

```bash
sudo systemctl restart containerd
```

---

## kubelet Not Running

```bash
systemctl status kubelet
```

Logs

```bash
journalctl -u kubelet -xe
```

---

## Firewall Issues

Check

Port

6443

is reachable from Worker Nodes.

---

# Part 2 Summary

Congratulations!

By completing this part, you have:

- Installed containerd
- Installed kubeadm
- Installed kubelet
- Installed kubectl
- Initialized the Control Plane
- Configured kubectl
- Installed the Flannel CNI
- Joined Worker Nodes
- Verified that the cluster is operational

Your Kubernetes cluster is now ready to deploy applications.


---

# 21. Production Best Practices

Installing a Kubernetes cluster is easy.

Maintaining a production cluster is the real challenge.

Follow these best practices.

---

## Use Multiple Control Plane Nodes

Production clusters should never have a single Master Node.

Recommended

```
3 Control Plane Nodes

+

3 or more Worker Nodes
```

Advantages

- High Availability
- No Single Point of Failure
- Better Reliability

---

## Use Odd Number of Control Plane Nodes

Recommended

```
1

3

5
```

Why?

Because **etcd** uses the Raft Consensus Algorithm.

Majority is required.

Example

```
3 Masters

↓

2 Required

↓

Cluster Healthy
```

---

## Backup etcd Regularly

etcd stores:

- Cluster State
- Secrets
- ConfigMaps
- Nodes
- Pods
- Deployments
- RBAC

Without etcd backup

↓

Cluster Recovery becomes difficult.

---

## Keep Kubernetes Versions Consistent

Avoid large version gaps.

Example

```
Master

1.34

Worker

1.30
```

Not Recommended.

---

## Secure API Server

Never expose the Kubernetes API Server publicly without proper security.

Use

- RBAC
- TLS Certificates
- Authentication
- Authorization
- Network Policies

---

## Monitor Cluster Health

Recommended Tools

- Prometheus
- Grafana
- Metrics Server

Monitor

- CPU
- Memory
- Storage
- Node Health
- Pod Health

---

## Enable Logging

Popular Logging Stack

```
Pods

↓

Fluent Bit

↓

Elasticsearch

↓

Kibana
```

---

## Keep Nodes Updated

Regularly patch

- Linux
- containerd
- kubelet
- kubeadm

---

# 22. Common Mistakes

## Mistake 1

Forgetting to disable swap.

Result

```
kubelet

↓

Fails
```

---

## Mistake 2

Not installing a CNI Plugin.

Result

```
Node

↓

NotReady
```

---

## Mistake 3

Ignoring Version Compatibility.

Always verify

```
Kubernetes

↓

containerd

↓

CNI
```

Compatibility.

---

## Mistake 4

Opening Kubernetes API Server to the Internet.

Very Dangerous.

---

## Mistake 5

Ignoring etcd Backup.

Cluster failure

↓

No Recovery.

---

# 23. Troubleshooting Guide

---

## Problem

```
kubectl get nodes

↓

NotReady
```

Check

```bash
kubectl describe node master
```

---

## Problem

Pods stuck in

```
Pending
```

Check

```bash
kubectl describe pod POD_NAME
```

Possible Causes

- No Resources
- Missing CNI
- Node Selector
- Taints

---

## Problem

ImagePullBackOff

Check

```bash
kubectl describe pod POD_NAME
```

Possible Reasons

- Wrong Image
- Docker Hub Rate Limit
- Private Registry Authentication

---

## Problem

CrashLoopBackOff

Check Logs

```bash
kubectl logs POD_NAME
```

---

## Problem

Worker Not Joining

Check

```bash
systemctl status kubelet
```

Verify

- Token
- Hash
- Port 6443
- Network Connectivity

---

## Useful Troubleshooting Commands

```bash
kubectl get nodes
```

```bash
kubectl get pods -A
```

```bash
kubectl describe node NODE_NAME
```

```bash
kubectl describe pod POD_NAME
```

```bash
kubectl logs POD_NAME
```

```bash
journalctl -u kubelet -xe
```

```bash
systemctl status containerd
```

```bash
kubectl cluster-info
```

---

# 24. Hands-on Lab

## Lab 1

Create

```
1 Master

2 Workers
```

Verify

```bash
kubectl get nodes
```

---

## Lab 2

Stop kubelet.

```bash
sudo systemctl stop kubelet
```

Observe

```
Node Status
```

Restart

```bash
sudo systemctl start kubelet
```

---

## Lab 3

Stop containerd.

Observe

Cluster Behavior.

Restart

```bash
sudo systemctl restart containerd
```

---

## Lab 4

Delete Flannel.

Observe

```
Node

↓

NotReady
```

Install Again.

---

## Lab 5

Reboot Worker Node.

Observe

- Node Status
- Pods
- Recovery Time

---

# 25. CKA Exam Tips

✔ Disable Swap First

✔ Learn kubeadm Commands

✔ Remember kubectl Configuration

✔ Understand CNI Installation

✔ Practice Worker Join

✔ Know Cluster Verification Commands

✔ Practice Troubleshooting

✔ Don't Memorize

Understand Every Command.

---

# 26. Interview Questions

## Basic

1. What is kubeadm?

2. Why do we use kubeadm?

3. Difference between kubeadm and kubectl?

4. Why disable swap?

5. What is containerd?

6. What is kubelet?

7. What is kubectl?

8. What is Flannel?

9. What is CNI?

10. Why does the node show NotReady?

---

## Intermediate

1. Explain kubeadm init.

2. Explain kubeadm join.

3. Why SystemdCgroup=true?

4. Explain kubelet Architecture.

5. Explain Cluster Bootstrapping.

6. Difference between CRI and CNI.

7. Explain Cluster Initialization Process.

8. What happens during kubeadm init?

9. How do Workers register?

10. Explain kubeconfig.

---

## Advanced

1. Explain Kubernetes Bootstrap Process.

2. Explain TLS Bootstrapping.

3. How are certificates generated?

4. How does kubelet communicate?

5. Explain API Server Authentication.

6. Explain Worker Registration.

7. Explain Control Plane Initialization.

8. Explain Cluster Recovery.

9. Explain etcd Backup Strategy.

10. Explain High Availability Installation.

---

## Scenario Based

### Scenario 1

Worker Node shows

```
NotReady
```

How will you troubleshoot?

---

### Scenario 2

Control Plane rebooted.

Cluster not working.

Where will you start?

---

### Scenario 3

Pods stuck in

```
Pending
```

How do you debug?

---

### Scenario 4

Developer says

```
kubectl not working
```

What will you verify?

---

### Scenario 5

Worker cannot join Cluster.

What could be the reasons?

---

# 27. Quick Revision

Cluster Setup Flow

```
Prepare Linux

↓

Disable Swap

↓

Load Kernel Modules

↓

Configure sysctl

↓

Install containerd

↓

Install kubeadm

↓

kubeadm init

↓

Configure kubectl

↓

Install Flannel

↓

Join Worker

↓

Verify Cluster
```

---

# 28. Commands Cheat Sheet

Update Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
```

Disable Swap

```bash
sudo swapoff -a
```

Load Modules

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Apply sysctl

```bash
sudo sysctl --system
```

Restart containerd

```bash
sudo systemctl restart containerd
```

Initialize Cluster

```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

Configure kubectl

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install Flannel

```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Verify Nodes

```bash
kubectl get nodes
```

View System Pods

```bash
kubectl get pods -n kube-system
```

Cluster Info

```bash
kubectl cluster-info
```

---

# 29. Key Takeaways

- kubeadm is the official Kubernetes cluster bootstrap tool.
- Proper Linux preparation is essential before installation.
- Disabling swap is mandatory for kubelet.
- containerd is the recommended container runtime.
- Flannel (or another CNI plugin) is required for pod networking.
- Worker nodes join the cluster using a secure bootstrap token.
- Always verify cluster health after installation.
- Regular backups of etcd and version management are critical in production.

---

# 30. Chapter Summary

Congratulations! 🎉

You have successfully learned how to build a Kubernetes cluster using **kubeadm**.

In this chapter, you covered:

- Preparing Linux servers
- Configuring the operating system
- Installing containerd
- Installing Kubernetes components
- Initializing the Control Plane
- Configuring kubectl
- Installing a CNI plugin
- Joining Worker Nodes
- Verifying the cluster
- Troubleshooting common issues
- Production best practices
- CKA exam tips
- Interview preparation

This completes the foundation of Kubernetes cluster installation. In the next chapter, you'll begin working with the cluster using **kubectl** and deploy your first applications.