# Kubernetes Class 04 - kubectl & Kubernetes Cluster Verification

> **Class Date:** 21-Jun-2026
>
> **Module:** kubectl Fundamentals & Cluster Verification
>
> **File Name:** `K8S-04_kubectl-and-Cluster-Verification_21-Jun-2026.md`

---

# 📚 Table of Contents

1. Introduction
2. What is kubectl?
3. kubectl Architecture
4. How kubectl Communicates with the Cluster
5. kubeconfig File
6. Contexts, Clusters and Users
7. Verifying the Kubernetes Cluster
8. Essential kubectl Commands
9. Understanding Kubernetes Namespaces
10. Understanding Kubernetes Resources
11. Output Formats
12. Useful kubectl Options
13. Production Tips
14. Common Errors
15. Troubleshooting
16. Interview Questions
17. Summary

---

# 1. Introduction

After installing a Kubernetes cluster using **kubeadm**, the next step is learning how to interact with it.

Kubernetes is managed using a command-line tool called **kubectl**.

Almost every Kubernetes administrator uses `kubectl` daily for:

- Creating resources
- Updating applications
- Monitoring workloads
- Troubleshooting issues
- Managing clusters

Without `kubectl`, administering Kubernetes would be extremely difficult.

---

# 2. What is kubectl?

`kubectl` is the official Kubernetes Command Line Interface (CLI).

It allows users to communicate with the Kubernetes API Server.

Think of it as a **remote control** for your Kubernetes cluster.

---

## Real World Analogy

Imagine a TV.

- TV → Kubernetes Cluster
- Remote → kubectl

Without the remote, controlling the TV becomes inconvenient.

Similarly, without `kubectl`, managing Kubernetes becomes much harder.

---

# 3. kubectl Architecture

```
Developer

      │

      ▼

  kubectl

      │

      ▼

API Server

      │

      ▼

Control Plane

      │

      ▼

Worker Nodes

      │

      ▼

Pods
```

Every kubectl command is sent to the **API Server**.

The API Server validates the request and updates the desired state.

---

# 4. How kubectl Works

Example command:

```bash
kubectl get pods
```

Execution Flow

```
kubectl

↓

Reads kubeconfig

↓

Connects to API Server

↓

Authenticates User

↓

Authorization Check

↓

Reads etcd

↓

Returns Response
```

---

# 5. kubeconfig File

`kubectl` needs connection details.

These are stored in a file called:

```text
~/.kube/config
```

This file contains:

- Cluster Information
- User Credentials
- Certificates
- Contexts

Without this file:

```
kubectl

↓

Cannot connect
```

---

## View kubeconfig

```bash
kubectl config view
```

---

## Current Context

```bash
kubectl config current-context
```

---

# 6. Contexts

A kubeconfig file may contain multiple clusters.

Example:

```
Development

Testing

Production
```

Instead of editing the configuration manually,

use contexts.

---

## List Contexts

```bash
kubectl config get-contexts
```

---

## Switch Context

```bash
kubectl config use-context production
```

---

# 7. Verify Cluster

First command after installation.

```bash
kubectl cluster-info
```

Example Output

```
Kubernetes control plane is running

CoreDNS is running
```

---

## Verify Nodes

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
```

---

## Wide Output

```bash
kubectl get nodes -o wide
```

Shows

- Internal IP
- OS
- Kernel Version
- Container Runtime
- Kubernetes Version

---

# 8. Understanding Kubernetes Resources

Everything in Kubernetes is an object.

Examples

- Pods
- Deployments
- ReplicaSets
- Services
- ConfigMaps
- Secrets
- PersistentVolumes
- Jobs

List all API resources

```bash
kubectl api-resources
```

---

# 9. Working with Namespaces

Namespaces logically separate resources.

Default namespaces:

```bash
kubectl get namespaces
```

Output

```
default

kube-system

kube-public

kube-node-lease
```

---

## Purpose of Each Namespace

### default

User applications.

---

### kube-system

Core Kubernetes components.

Example

- CoreDNS
- kube-proxy
- etcd
- API Server

---

### kube-public

Publicly readable resources.

---

### kube-node-lease

Stores node heartbeat information.

---

# 10. Essential kubectl Commands

## Get Pods

```bash
kubectl get pods
```

---

## Get All Pods

```bash
kubectl get pods -A
```

---

## Get Services

```bash
kubectl get svc
```

---

## Get Deployments

```bash
kubectl get deployments
```

---

## Get ReplicaSets

```bash
kubectl get rs
```

---

## Get Events

```bash
kubectl get events
```

Useful while troubleshooting.

---

## Get Everything

```bash
kubectl get all
```

Shows

- Pods
- Services
- ReplicaSets
- Deployments

---

# 11. Describe Resources

The `get` command provides a summary.

`describe` provides detailed information.

Example

```bash
kubectl describe node master
```

Shows

- CPU
- Memory
- Conditions
- Labels
- Capacity
- Allocatable Resources

---

## Describe Pod

```bash
kubectl describe pod POD_NAME
```

Very useful for debugging.

---

# 12. Output Formats

Default

```bash
kubectl get pods
```

Wide

```bash
kubectl get pods -o wide
```

YAML

```bash
kubectl get pod POD_NAME -o yaml
```

JSON

```bash
kubectl get pod POD_NAME -o json
```

Custom Columns

```bash
kubectl get pods \
-o custom-columns=NAME:.metadata.name,STATUS:.status.phase
```

---

# 13. Useful kubectl Options

Watch resources continuously

```bash
kubectl get pods -w
```

Sort by creation time

```bash
kubectl get pods --sort-by=.metadata.creationTimestamp
```

Explain resource fields

```bash
kubectl explain pod
```

Explain container specification

```bash
kubectl explain pod.spec.containers
```

This command is extremely useful when writing YAML files.

---

# 14. Production Tips

✔ Always verify the current context before running commands.

```bash
kubectl config current-context
```

✔ Use namespaces to isolate applications.

✔ Prefer `kubectl describe` over guessing.

✔ Use `kubectl explain` while writing manifests.

✔ Avoid working directly in the `default` namespace for production workloads.

---

# 15. Common Errors

## Error

```
The connection to the server localhost:8080 was refused
```

Reason

- kubeconfig missing
- API Server unavailable

---

## Error

```
No resources found
```

Reason

- Wrong namespace
- No workloads deployed

---

## Error

```
Forbidden
```

Reason

RBAC permission issue.

---

## Error

```
Unable to connect to the server
```

Check

- API Server
- Network
- Certificates
- kubeconfig

---

# 16. Troubleshooting Commands

Check Cluster

```bash
kubectl cluster-info
```

Check Nodes

```bash
kubectl get nodes
```

Check System Pods

```bash
kubectl get pods -n kube-system
```

Describe Node

```bash
kubectl describe node master
```

View Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# 17. Interview Questions

## Basic

1. What is kubectl?
2. Where is the kubeconfig file stored?
3. What does `kubectl get nodes` display?
4. What is the purpose of namespaces?
5. Difference between `get` and `describe`?

---

## Intermediate

1. Explain how kubectl communicates with Kubernetes.
2. What is a context?
3. Explain kubeconfig structure.
4. What does `kubectl explain` do?
5. How do you switch between clusters?

---

## Scenario-Based

### Scenario 1

`kubectl` cannot connect to the cluster.

How would you troubleshoot?

---

### Scenario 2

A developer reports that no Pods are visible.

What commands would you run first?

---

### Scenario 3

You accidentally ran a command on the production cluster.

How could this have been avoided?

---

# 18. Quick Cheat Sheet

```bash
kubectl cluster-info
kubectl get nodes
kubectl get pods -A
kubectl get namespaces
kubectl get all
kubectl describe node master
kubectl describe pod POD_NAME
kubectl config view
kubectl config current-context
kubectl config get-contexts
kubectl explain pod
kubectl api-resources
```

---

# 19. Key Takeaways

- `kubectl` is the primary CLI for Kubernetes administration.
- It communicates with the Kubernetes API Server using the kubeconfig file.
- Namespaces help organize and isolate resources.
- `kubectl get` provides summaries, while `kubectl describe` gives detailed diagnostics.
- `kubectl explain` is invaluable for understanding Kubernetes resource specifications.
- Mastering these commands is the foundation for working with Pods, Deployments, Services, and all other Kubernetes resources.

---

# 20. Chapter Summary

In this chapter, you learned:

- What `kubectl` is and how it works
- The role of the kubeconfig file
- Contexts, clusters, and users
- Cluster verification after installation
- Essential `kubectl` commands
- Kubernetes namespaces
- Output formats
- Production tips
- Common troubleshooting techniques
- Interview-focused questions

This chapter prepares you for the next step: creating and managing your **first Pods** using Kubernetes manifests and imperative commands.