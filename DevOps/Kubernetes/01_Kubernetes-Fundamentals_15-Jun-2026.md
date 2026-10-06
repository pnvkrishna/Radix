# Kubernetes Class 01 - Introduction & Kubernetes Fundamentals
> **Class Date:** 15-Jun-2026
>
> **Module:** Kubernetes Fundamentals
>
> **Duration:** Day 1
>
> **Goal:** Understand the history of containers, why Kubernetes was created, Kubernetes architecture basics, and cluster components.

---

# Table of Contents

1. Introduction
2. Google's History with Containers
3. Evolution of Container Orchestration
4. What is Kubernetes?
5. Why Kubernetes?
6. CNCF
7. Kubernetes and Docker
8. Kubernetes Interfaces
   - CRI
   - CNI
   - CSI
9. Kubernetes Cluster
10. Nodes
11. Master Node
12. Worker Node
13. Kubernetes Architecture
14. Story of Phippy
15. Key Takeaways
16. Interview Questions

---

# Introduction

Modern applications are expected to be:

- Highly Available
- Scalable
- Fault Tolerant
  - the ability of a system to keep working without interruption when one or more of its parts fail.
- Easy to Deploy
- Easy to Upgrade

Managing thousands of application containers manually is almost impossible.

This is where **Kubernetes** comes into the picture.

Kubernetes automates:

- Deployment
- Scaling
- Networking
- Load Balancing
- Self Healing
- Rolling Updates
- Service Discovery

---

# Google's History with Containers

Many people think containers started with Docker.

**This is not true.**

Google has been running containers internally for more than **15 years** before Docker became popular.

Google manages billions of containers every week.

Some of Google's products running on containers include:

- Gmail
- Google Search
- YouTube
- Google Maps
- Google Drive

Google needed a system capable of managing millions of containers.

This led to the development of internal container orchestration systems.

---

# Evolution of Google's Container Platforms

## 1. Borg

Borg was Google's first large-scale container orchestration platform.

Features:

- Scheduling
- Resource Management
- Automatic Recovery
- High Availability

Borg was never open sourced.

---

## 2. Omega

Google later improved Borg and created Omega.

Omega introduced:

- Better Scheduling
- Parallel Scheduling
- Improved Cluster Management

Still, it remained an internal project.

---

## 3. Kubernetes

When Docker became popular around 2013-2014, Google decided to build an open-source container orchestration platform.

This platform became **Kubernetes**.

Google donated Kubernetes to the **Cloud Native Computing Foundation (CNCF)**.

Today Kubernetes is maintained by thousands of contributors worldwide.

---

# Why Was Kubernetes Created?

Imagine you have:

- 10 servers
- 500 Docker containers

Questions arise:

- Which server should run which container?
- What if a server crashes?
- What if a container dies?
- How do you scale applications?
- How do users access containers?

Managing all these manually becomes difficult.

Kubernetes solves these problems automatically.

---

# What is Kubernetes?

According to the official definition:

> Kubernetes is an open-source platform for automating deployment, scaling, and management of containerized applications.

Simple definition:

Kubernetes is the **Operating System for Containers.**

Just like Windows manages applications,

Kubernetes manages containers.

---

# Why is Kubernetes Called K8s?

"Kubernetes"

contains 10 letters.

Replace the middle eight letters with **8**

K + 8 + s

↓

K8s

---

# Kubernetes is Written in Go (Golang)

Programming Language:

- Go (Golang)

Reasons:

- Fast
- Lightweight
- Concurrent
- Portable
- Excellent Networking Support

---

# CNCF

CNCF stands for

**Cloud Native Computing Foundation**

CNCF maintains projects like:

- Kubernetes
- Prometheus
- Helm
- containerd
- Envoy
- Fluentd

Website:

https://www.cncf.io

---

# Kubernetes and Docker

Initially Kubernetes supported only Docker.

Architecture looked like this:

```
Application

↓

Docker

↓

Linux
```

Later Kubernetes wanted to support multiple container runtimes.

Instead of talking directly to Docker,

it introduced interfaces.

---

# Kubernetes Interfaces

Kubernetes communicates with three major components.

```
          Kubernetes

      /        |        \

    CRI       CNI      CSI
```

---

# 1. CRI

Container Runtime Interface

Purpose:

Communication between Kubernetes and the Container Runtime.

Examples:

- containerd
- CRI-O
- Docker (via cri-dockerd)

Responsibilities:

- Start Containers
- Stop Containers
- Delete Containers
- Pull Images

---

# 2. CNI

Container Network Interface

Purpose:

Networking for Pods.

Responsibilities:

- Assign IP Address
- Connect Pods
- Pod Communication
- Network Policies

Popular CNI Plugins:

- Calico
- Flannel
- Cilium
- Weave
- Canal

Important:

Kubernetes does **NOT** install a default CNI plugin.

Administrator must install one.

Without CNI:

Pods remain in

```
NotReady
```

state.

---

# 3. CSI

Container Storage Interface

Purpose:

Persistent Storage.

Responsibilities:

- Attach Volumes
- Mount Storage
- Dynamic Provisioning
- Persistent Volumes

Examples:

- AWS EBS CSI Driver
- Azure Disk CSI
- GCE Persistent Disk CSI
- NFS CSI
- Ceph CSI

---

# Kubernetes Cluster

A Kubernetes Cluster is a collection of computers working together.

It consists of:

- Master Node(s)
- Worker Node(s)

Example:

```
                Kubernetes Cluster

        +---------------------------+

        |      Master Node          |

        +---------------------------+

             /        |         \

      Worker-1   Worker-2   Worker-3
```

---

# Node

A Node is a machine that participates in the cluster.

A node can be:

- Physical Server
- Virtual Machine
- Cloud Instance

Examples:

- AWS EC2
- Azure VM
- Google Compute Engine

---

# Types of Nodes

## Master Node

Responsibilities:

- Cluster Management
- Scheduling
- API Handling
- Authentication
- Controller Management

Master Node controls everything.

---

## Worker Node

Worker Nodes run applications.

Responsibilities:

- Run Pods
- Pull Images
- Mount Storage
- Execute Containers

Users' applications always run on Worker Nodes.

---

# Kubernetes Architecture Overview

```
             kubectl

                |

                |

          API Server

                |

------------------------------------------

|              |              |

Scheduler   Controller     etcd

------------------------------------------

         Worker Nodes

     Pod    Pod    Pod
```

---

# Story of Phippy

Phippy is the official Kubernetes learning mascot.

It was created to explain cloud-native technologies in a simple and fun way.

Books include:

- The Children's Illustrated Guide to Kubernetes
- Phippy Goes to the Zoo
- Phippy and Friends

Phippy helps beginners understand Kubernetes concepts visually.

---

# Key Takeaways

- Google has been using containers for many years.
- Borg and Omega inspired Kubernetes.
- Kubernetes is open source.
- Kubernetes is maintained by CNCF.
- Kubernetes is written in Go.
- Kubernetes manages containerized applications.
- Kubernetes communicates using CRI, CNI, and CSI.
- A Kubernetes cluster contains Master and Worker nodes.
- Kubernetes does not include a default CNI plugin.

---

# Interview Questions

## Basic

1. What is Kubernetes?
2. Why was Kubernetes created?
3. What is Borg?
4. What is Omega?
5. Why is Kubernetes called K8s?
6. Which language is Kubernetes written in?
7. What is CNCF?
8. What is a Kubernetes Cluster?
9. What is a Node?
10. Difference between Master Node and Worker Node?

---

## Intermediate

1. Explain CRI.
2. Explain CNI.
3. Explain CSI.
4. Why doesn't Kubernetes include a default CNI?
5. Can Kubernetes work without Docker?
6. What is containerd?
7. What happens if CNI is missing?
8. Why was Docker support removed from Kubernetes?
9. Explain Kubernetes architecture.
10. Explain cluster components.

---

# Summary

In this class, we learned:

- Google's container journey
- Borg → Omega → Kubernetes evolution
- Why Kubernetes was created
- CNCF and open-source governance
- Kubernetes architecture at a high level
- CRI, CNI, and CSI interfaces
- Master and Worker Nodes
- Kubernetes Cluster fundamentals

This knowledge forms the foundation for all future Kubernetes topics.