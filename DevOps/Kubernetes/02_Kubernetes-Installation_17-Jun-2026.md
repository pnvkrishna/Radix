# Kubernetes Class 02 - Kubernetes Installation Options
> **Class Date:** 17-Jun-2026
>
> **Module:** Kubernetes Installation

---

# Table of Contents

1. Introduction
2. Kubernetes Installation Overview
3. Single Node vs Multi Node Cluster
4. Local Kubernetes Solutions
5. Multi Node Kubernetes
6. Self Managed vs Managed Kubernetes
7. Kubernetes as a Service
8. Master Node Requirements
9. Worker Node Requirements
10. Kubernetes Installation Methods
11. Container Runtime (CRI)
12. Container Network Interface (CNI)
13. Kubernetes Installation Workflow
14. Lab Overview
15. Best Practices
16. Interview Questions
17. Summary

---

# Introduction

Before learning Kubernetes workloads such as Pods, Deployments, and Services, we first need a Kubernetes cluster.

There are many ways to install Kubernetes.

The installation method depends on:

- Learning
- Development
- Testing
- Production
- Enterprise Requirements
- Budget

There is **no single installation method** that fits every use case.

---

# Kubernetes Installation Overview

Broadly Kubernetes installation falls into two categories.

```
                Kubernetes

          /                    \

   Single Node             Multi Node
```

---

# Single Node Kubernetes

Single-node Kubernetes is mainly used for:

- Learning Kubernetes
- Practice
- Local Development
- Testing YAML files
- CI/CD Testing

Advantages

- Easy setup
- Less RAM
- Quick installation
- Easy cleanup

Disadvantages

- Not Highly Available
- Not suitable for Production
- Limited scalability

---

# Popular Single Node Solutions

## 1. Kind

Kind stands for

**Kubernetes IN Docker**

Kind creates Kubernetes nodes as Docker containers.

Architecture

```
Laptop

↓

Docker

↓

Kind

↓

Kubernetes Cluster
```

Advantages

- Very lightweight
- Fast cluster creation
- Excellent for CI/CD
- Used by Kubernetes developers

Use Cases

- Learning
- GitHub Actions
- Jenkins Pipelines
- Automated Testing

---

## 2. Minikube

Minikube runs a complete Kubernetes cluster locally using a VM or container runtime.

Architecture

```
Laptop

↓

Docker / Hypervisor

↓

Minikube

↓

Kubernetes Cluster
```

Advantages

- Beginner Friendly
- Dashboard Support
- Multiple Drivers
- Easy Installation

Recommended For

- Beginners
- Students
- Developers

---

# Which One Should You Choose?

| Tool | Best For | Difficulty |
|-------|----------|------------|
| Kind | CI/CD, Developers | Easy |
| Minikube | Beginners | Very Easy |

---

# Multi Node Kubernetes

Production applications rarely run on one server.

Instead, multiple nodes are connected together.

Example

```
             Kubernetes Cluster

         Master Node

        /      |      \

 Worker1  Worker2  Worker3
```

Benefits

- High Availability
- Scalability
- Load Distribution
- Fault Tolerance

---

# Multi Node Deployment Options

There are two major approaches.

```
             Multi Node

        /                  \

Self Managed        Managed Service
```

---

# Self Managed Kubernetes

In this approach,

you manage everything.

You create:

- Servers
- Networking
- Security
- Kubernetes Installation
- Upgrades
- Backup

Servers can be

- Physical
- VMware
- Virtual Machines
- Cloud VMs

Examples

- Azure Virtual Machines
- AWS EC2
- Google Compute Engine
- On-premise Servers

Advantages

- Full Control
- Low Cost
- Custom Configuration

Disadvantages

- More Maintenance
- Upgrade Responsibility
- Security Responsibility

---

# Managed Kubernetes

Cloud providers manage the Kubernetes Control Plane.

You manage only:

- Applications
- Pods
- Deployments
- Services

Examples

## Amazon EKS

Elastic Kubernetes Service

AWS manages:

- API Server
- etcd
- Scheduler

You manage:

- Worker Nodes
- Applications

---

## Azure AKS

Azure Kubernetes Service

Microsoft manages the Control Plane.

---

## Google GKE

Google Kubernetes Engine

Created by Google.

One of the most mature Kubernetes offerings.

---

# Which Option Should I Learn?

| Purpose | Recommended |
|----------|-------------|
| Learning | Minikube |
| Daily Practice | Kind |
| Dev Environment | Kind / kubeadm |
| Production | kubeadm |
| Enterprise Cloud | AKS / EKS / GKE |

---

# Kubernetes Nodes

Every Kubernetes cluster contains Nodes.

```
Cluster

↓

Nodes

↓

Pods
```

Nodes may be

- Physical Machines
- Virtual Machines
- Cloud Instances

---

# Master Node

The Master Node controls the cluster.

Responsibilities

- Scheduling
- Authentication
- API Requests
- Cluster State
- Controller Management

Master Nodes should always run Linux.

Reason

Most Kubernetes control plane components are designed primarily for Linux environments.

---

# Worker Node

Worker Nodes execute workloads.

Responsibilities

- Running Pods
- Pulling Images
- Executing Containers
- Connecting Storage

Worker Nodes can be

- Linux
- Windows

Linux is most commonly used.

---

# Kubernetes Installation Methods

There are several ways to install Kubernetes.

| Method | Use Case |
|---------|----------|
| Kind | Learning |
| Minikube | Learning |
| kubeadm | Production |
| Kubespray | Production |
| RKE2 | Enterprise |
| OpenShift | Enterprise |
| K3s | Edge Computing |

---

# kubeadm

kubeadm is the official Kubernetes cluster bootstrapping tool.

It helps create a production-like Kubernetes cluster.

Typical workflow

```
Servers

↓

Install Container Runtime

↓

Install kubeadm

↓

Initialize Cluster

↓

Install CNI

↓

Join Worker Nodes
```

---

# CRI (Container Runtime Interface)

Kubernetes itself cannot run containers.

It communicates with a Container Runtime through CRI.

Examples

- containerd
- CRI-O
- cri-dockerd

Responsibilities

- Pull Images
- Create Containers
- Stop Containers
- Delete Containers

```
Kubernetes

↓

CRI

↓

containerd
```

---

# CNI (Container Network Interface)

Pods need networking.

CNI provides:

- Pod Networking
- Pod-to-Pod Communication
- IP Allocation
- Network Policies

Popular Plugins

- Flannel
- Calico
- Cilium
- Weave Net

Important

Kubernetes does **NOT** install a default CNI.

Without a CNI plugin, Pods cannot communicate correctly and cluster networking remains incomplete. :contentReference[oaicite:1]{index=1}

---

# Lab Overview

Lab Environment

```
Node 1

Master

Ubuntu

↓

Node 2

Worker

Ubuntu
```

Cloud Platform

Azure Virtual Machines

(You can also use AWS or GCP.)

Steps

1. Create two Ubuntu Virtual Machines
2. Install Docker (or another supported container runtime)
3. Install CRI support if using Docker
4. Install kubeadm
5. Initialize Master
6. Install CNI
7. Join Worker Node
8. Verify Cluster

---

# Installation Flow

```
Create VMs

↓

Install Container Runtime

↓

Install kubeadm

↓

kubeadm init

↓

Install CNI

↓

Join Worker

↓

Verify Cluster
```

---

# Best Practices

✔ Use Linux for Control Plane Nodes

✔ Keep Kubernetes versions consistent

✔ Choose a CNI before cluster initialization

✔ Learn kubeadm before managed Kubernetes

✔ Practice on local clusters before production

✔ Understand CRI and CNI before installation

---

# Interview Questions

## Basic

1. What are the different ways to install Kubernetes?
2. What is Kind?
3. What is Minikube?
4. What is kubeadm?
5. Difference between Single Node and Multi Node Cluster?
6. What is AKS?
7. What is EKS?
8. What is GKE?
9. What is CRI?
10. What is CNI?

---

## Intermediate

1. Why should beginners start with Minikube?
2. Why is kubeadm used in production?
3. Explain the kubeadm installation workflow.
4. What happens if no CNI is installed?
5. Can Kubernetes run without Docker?
6. Why is containerd preferred today?
7. Explain Self Managed vs Managed Kubernetes.
8. Why is Linux preferred for Control Plane nodes?
9. How do Worker Nodes join a cluster?
10. What components are managed by cloud providers in AKS/EKS/GKE?

---

# Key Takeaways

- Kubernetes installation depends on the use case.
- Single-node clusters are ideal for learning and development.
- Multi-node clusters are designed for production.
- Self-managed clusters provide complete control but require more operational effort.
- Managed Kubernetes services reduce infrastructure management.
- kubeadm is the standard tool for building production-like clusters.
- Kubernetes depends on CRI for container runtimes and CNI for networking.
- Installing a CNI plugin is an essential step after cluster initialization.

---

# Summary

In this class, we explored the various ways to install Kubernetes, from lightweight local environments such as Kind and Minikube to production-ready clusters using kubeadm and managed services like AKS, EKS, and GKE. We also introduced the concepts of CRI, CNI, and the overall cluster installation workflow. These concepts lay the foundation for the next class, where we'll build a Kubernetes cluster using **kubeadm**.