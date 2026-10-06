# Kubernetes Class 05 - Pods (The Smallest Deployable Unit)

> **Class Date:** 27-Jun-2026
>
> **Module:** Pods


---

# 📚 Table of Contents

1. Introduction
2. Why Pods?
3. What is a Pod?
4. Why Kubernetes Doesn't Run Containers Directly
5. Pod Architecture
6. Pod Lifecycle
7. Pod Phases
8. Single Container Pod
9. Multi Container Pod
10. Pod Networking
11. Pause Container
12. Pod vs Container
13. Production Best Practices
14. Summary

---

# 1. Introduction

Now that our Kubernetes cluster is ready, it's time to deploy our first application.

Before learning Deployments, ReplicaSets, Services, or Ingress, you must understand **Pods**, because **every application in Kubernetes runs inside a Pod**.

Pods are the smallest unit that Kubernetes creates, schedules, and manages.

Think of a Pod as the basic building block of Kubernetes.

---

# 2. Why Pods?

A common question is:

> "Why doesn't Kubernetes run Docker containers directly?"

The answer lies in Kubernetes' design.

Containers are created and managed by a **Container Runtime** (such as containerd), while Kubernetes manages **Pods**.

Pods provide an additional layer that offers:

- Shared networking
- Shared storage
- Lifecycle management
- Health monitoring
- Scheduling
- Resource management

This design gives Kubernetes more flexibility than managing containers directly.

---

# Real World Analogy

Imagine an apartment building.

- **Building** → Kubernetes Cluster
- **Floor** → Node
- **Apartment** → Pod
- **Person living inside** → Container

Kubernetes allocates apartments (Pods), and the containers live inside them.

It never allocates people directly.

---

# 3. What is a Pod?

According to the Kubernetes documentation:

> A Pod is the smallest deployable unit in Kubernetes.

A Pod is a wrapper around one or more containers.

It provides:

- Networking
- Storage
- Configuration
- Lifecycle

A Pod may contain:

- One Container ✅ (Most common)
- Multiple Containers ✅ (Special cases)

---

# Pod Architecture

```
          Kubernetes Node

+--------------------------------------+

           Pod

+--------------------------------------+

      Container

      nginx

+--------------------------------------+

Container Runtime

+--------------------------------------+

Linux Kernel
```

---

# 4. Why Kubernetes Doesn't Run Containers Directly

Imagine an application consisting of:

- Web Server
- Logging Agent
- Monitoring Agent

If Kubernetes managed containers individually:

```
Container A

Container B

Container C
```

Managing networking and storage would become complicated.

Instead, Kubernetes groups them into a Pod.

```
             Pod

+---------------------------+

Container A

Container B

Container C

+---------------------------+
```

Now they can:

- Share IP Address
- Share Storage
- Start Together
- Stop Together

---

# 5. Pod Architecture

```
                  Kubernetes Cluster

                         │

                    Scheduler

                         │

                         ▼

                     Worker Node

                         │

                         ▼

                    +-----------+
                    |    Pod    |
                    +-----------+
                    | Container |
                    | Pause     |
                    +-----------+
```

Every Pod contains at least:

- One Application Container
- One Pause Container

Many beginners don't know about the Pause Container.

We'll discuss it shortly.

---

# Components Inside a Pod

A Pod may contain:

- Application Container
- Init Container
- Sidecar Container
- Pause Container
- Shared Volumes

Example

```
             Pod

+--------------------------------+

Pause Container

+

Application Container

+

Sidecar Container

+

Shared Volume

+--------------------------------+
```

---

# 6. Pod Lifecycle

A Pod goes through several phases.

```
Created

↓

Scheduled

↓

Pulled Image

↓

Container Created

↓

Running

↓

Succeeded

or

Failed
```

Understanding the Pod lifecycle is essential for troubleshooting.

---

# Detailed Pod Lifecycle

## Step 1

User creates Pod.

```
kubectl apply

↓

API Server
```

---

## Step 2

Scheduler selects a Worker Node.

```
Scheduler

↓

Worker Node Selected
```

---

## Step 3

kubelet receives instructions.

```
API Server

↓

kubelet
```

---

## Step 4

containerd pulls the image.

Example

```
nginx

↓

Docker Hub
```

---

## Step 5

Container starts.

```
Running
```

---

# Complete Workflow

```
kubectl apply

↓

API Server

↓

Scheduler

↓

Worker Node

↓

kubelet

↓

containerd

↓

Docker Hub

↓

Image Downloaded

↓

Container Started

↓

Pod Running
```

---

# 7. Pod Phases

Pods have five official phases.

## Pending

The Pod has been accepted but is not yet running.

Reasons

- Image Download
- Scheduling
- Volume Mounting

---

## Running

The Pod is successfully running.

At least one container is active.

---

## Succeeded

All containers completed successfully.

Usually seen with Jobs.

---

## Failed

One or more containers terminated unexpectedly.

Reasons

- Application Crash
- Image Problems
- Configuration Errors

---

## Unknown

The Kubernetes control plane cannot determine the Pod state.

Usually caused by:

- Network Issues
- Node Failure

---

# Check Pod Status

```bash
kubectl get pods
```

Example

```
NAME

nginx

STATUS

Running
```

---

# Describe Pod

```bash
kubectl describe pod nginx
```

Shows

- Events
- Node
- Image
- Labels
- Conditions
- Container Status
- Volumes
- Restart Count

---

# Get Pod in YAML

```bash
kubectl get pod nginx -o yaml
```

This command is extremely useful.

It displays the complete Pod definition generated by Kubernetes.

---

# 8. Single Container Pod

Most Pods contain only one container.

Example

```
Pod

↓

nginx
```

Benefits

- Simpler
- Easier Monitoring
- Easier Scaling
- Easier Troubleshooting

Production Recommendation

✔ Prefer one container per Pod unless there is a strong architectural reason to use multiple containers.

---

# Example Architecture

```
Node

↓

Pod

↓

Nginx Container
```

---

# Why One Container Per Pod?

Because:

- Independent Scaling
- Better Fault Isolation
- Easier Updates
- Simpler Monitoring

This follows cloud-native best practices.

---

# Part 1 Summary

In this section, you learned:

- What a Pod is
- Why Pods exist
- Why Kubernetes manages Pods instead of containers
- Pod architecture
- Components inside a Pod
- Pod lifecycle
- Pod phases
- Single-container Pods

These concepts are the foundation for deploying applications in Kubernetes.

---

# 9. Multi-Container Pods

Until now, we have learned that most Pods contain a single container.

However, Kubernetes also supports multiple containers inside the same Pod.

Example:

```
+---------------------------------------------------+
|                     POD                           |
|---------------------------------------------------|
|                                                   |
|  Nginx Container                                  |
|                                                   |
|  Log Collector (Sidecar)                          |
|                                                   |
|  Shared Volume                                    |
|                                                   |
+---------------------------------------------------+
```

Both containers

- Share the same IP Address
- Share the same Network Namespace
- Can communicate using **localhost**
- Can share storage volumes

---

## Why Use Multiple Containers?

Suppose your application generates logs.

Instead of sending logs directly,

you can run another container.

Example

```
Pod

↓

Application Container

+

Fluent Bit Container

↓

ElasticSearch
```

Application only writes logs.

Sidecar collects logs.

---

## Real Production Example

Application

↓

Writes logs

↓

Sidecar Container

↓

Sends logs to ELK

Examples

- Fluent Bit
- Fluentd
- Vector

---

## Other Examples

Application

↓

Prometheus Exporter

Application

↓

Security Scanner

Application

↓

Log Forwarder

---

## When Should We Use Multiple Containers?

Use only when

Containers

- Must work together
- Must share storage
- Must share network

Otherwise

Use separate Pods.

---

# 10. Pause Container

One of the most important Kubernetes concepts.

Many DevOps Engineers don't know this.

Every Pod contains

```
Pause Container
```

---

## Why?

Imagine Kubernetes starts nginx.

Question

Who owns

- Network Namespace?

- IPC Namespace?

- PID Namespace?

Answer

Pause Container.

---

## Architecture

```
                POD

+----------------------------------+

Pause Container

↓

Network

↓

Storage

↓

Namespaces

+

Application Container

+

Sidecar

+----------------------------------+
```

---

## Responsibilities

Pause Container creates

- Network Namespace

- IPC Namespace

- PID Namespace

Application Containers join it.

---

## Real Example

Check

```bash
crictl ps
```

or

```bash
docker ps
```

You will see

```
pause

nginx
```

---

# 11. Pod Networking

One of Kubernetes' biggest strengths.

Every Pod receives

```
One Unique IP Address
```

Example

```
Pod A

10.244.0.2
```

```
Pod B

10.244.0.3
```

Communication

```
10.244.0.2

↓

10.244.0.3
```

No NAT required.

---

## Kubernetes Networking Rules

Every Pod

↓

Gets Unique IP

Every Pod

↓

Can Communicate

Every Node

↓

Can Reach Every Pod

These rules are implemented using

```
CNI
```

---

## Communication

```
Pod A

↓

Pod B

↓

Service

↓

Internet
```

---

## Verify Pod IP

```bash
kubectl get pods -o wide
```

Example

```
NAME

nginx

IP

10.244.0.8
```

---

# 12. Labels

Labels are key-value pairs.

Example

```
app=nginx

env=dev

tier=frontend
```

Labels help Kubernetes identify objects.

---

## Example

```
Pod

↓

app=nginx

↓

version=v1

↓

environment=production
```

---

## Why Labels?

Imagine

500 Pods.

How do you identify

Frontend Pods?

Answer

Labels.

---

## View Labels

```bash
kubectl get pods --show-labels
```

---

# 13. Selectors

Selectors search using labels.

Example

```
app=nginx
```

Returns

All nginx Pods.

---

## Architecture

```
Service

↓

Selector

↓

app=nginx

↓

Pod1

Pod2

Pod3
```

Without labels,

Services cannot find Pods.

---

# 14. Imperative vs Declarative

Kubernetes supports two methods.

---

## Imperative

You tell Kubernetes

exactly

what to do.

Example

```bash
kubectl run nginx \
--image=nginx
```

Advantages

- Fast

- Easy

Disadvantages

- Difficult to Track

- Not Version Controlled

---

## Declarative

Write YAML.

Example

```bash
kubectl apply -f pod.yaml
```

Advantages

- Version Control

- GitOps

- Easy Rollback

Production always prefers

```
Declarative
```

---

# 15. Creating Your First Pod

Imperative

```bash
kubectl run nginx \
--image=nginx
```

Verify

```bash
kubectl get pods
```

Output

```
NAME

nginx

STATUS

Running
```

---

# 16. Pod YAML

Simple Example

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: nginx

spec:

  containers:

  - name: nginx

    image: nginx
```

---

# Line-by-Line Explanation

## apiVersion

Defines Kubernetes API version.

```
v1
```

---

## kind

Defines resource type.

```
Pod
```

---

## metadata

Stores information.

Example

```
name

labels

annotations
```

---

## spec

Desired State.

Everything Kubernetes should create.

---

## containers

List of Containers.

---

## name

Container Name.

---

## image

Container Image.

Example

```
nginx
```

↓

Pulled from Docker Hub.

---

# Apply YAML

```bash
kubectl apply -f pod.yaml
```

---

# Verify

```bash
kubectl get pods
```

---

# Describe

```bash
kubectl describe pod nginx
```

---

# Delete

```bash
kubectl delete pod nginx
```

---

# 17. kubectl apply

Most important command.

```
kubectl apply
```

Unlike

```
kubectl create
```

it

Creates

or

Updates

Resources.

Production teams almost always use

```
kubectl apply
```

---

# 18. Common Beginner Mistakes

❌ Forgetting YAML indentation.

❌ Wrong image name.

❌ Missing metadata.

❌ Wrong apiVersion.

❌ Editing running Pods directly.

❌ Not checking Events.

---

# Part 2 Summary

Congratulations!

Now you understand

- Multi-Container Pods
- Pause Container
- Pod Networking
- Labels
- Selectors
- Imperative Commands
- Declarative YAML
- kubectl apply
- First Pod Deployment

You can now deploy applications into Kubernetes using both imperative and declarative approaches.


---

# 19. Pod Restart Policies

A Restart Policy defines what Kubernetes should do when a container inside a Pod exits.

Kubernetes supports three restart policies:

| Restart Policy | Description |
|----------------|-------------|
| Always | Restart the container whenever it stops (Default) |
| OnFailure | Restart only if the container exits with an error |
| Never | Never restart the container |

---

## Always (Default)

```yaml
restartPolicy: Always
```

Used for:

- Web Applications
- APIs
- Microservices

Workflow

```
Container Starts

↓

Application Crashes

↓

kubelet Detects Failure

↓

Container Restarted
```

---

## OnFailure

```yaml
restartPolicy: OnFailure
```

Used for:

- Batch Processing
- Jobs
- Data Processing Tasks

---

## Never

```yaml
restartPolicy: Never
```

Used for:

- Debugging
- One-time Execution
- Testing

---

# 20. Init Containers

Init Containers run **before** the main application container starts.

Their purpose is to prepare the environment.

Examples:

- Wait for a database
- Download configuration files
- Perform initialization tasks
- Create directories

---

## Workflow

```
Init Container

↓

Success

↓

Application Container Starts
```

If the Init Container fails, the application container will **not** start.

---

## Example YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  initContainers:
  - name: init-service
    image: busybox
    command: ['sh', '-c', 'echo Initializing... && sleep 5']

  containers:
  - name: nginx
    image: nginx
```

---

# 21. Static Pods

Normally, Pods are created by the Kubernetes API Server.

Static Pods are different.

They are managed directly by the **kubelet**.

---

## Where are Static Pod Manifests Stored?

Usually:

```text
/etc/kubernetes/manifests/
```

Examples:

- kube-apiserver
- etcd
- kube-controller-manager
- kube-scheduler

These are control plane components created as Static Pods in a kubeadm cluster.

---

## Verify Static Pods

```bash
kubectl get pods -n kube-system
```

You will see pods like:

```
kube-apiserver
etcd
kube-controller-manager
kube-scheduler
```

---

# 22. Pod Health

Kubernetes continuously monitors Pod health.

Useful commands:

```bash
kubectl get pods
```

```bash
kubectl describe pod POD_NAME
```

```bash
kubectl logs POD_NAME
```

---

## Common Pod Status Values

| Status | Meaning |
|---------|---------|
| Pending | Waiting for scheduling or resources |
| Running | Pod is active |
| Completed | Successfully finished |
| Error | Application exited unexpectedly |
| CrashLoopBackOff | Container repeatedly crashes |
| ImagePullBackOff | Unable to pull image |
| ErrImagePull | Image download failed |
| ContainerCreating | Pod is being created |
| Terminating | Pod is shutting down |

---

# 23. Pod Troubleshooting

One of the most important skills for a Kubernetes administrator.

---

## Step 1 - Check Pod Status

```bash
kubectl get pods
```

---

## Step 2 - Describe the Pod

```bash
kubectl describe pod POD_NAME
```

Look for:

- Events
- Scheduling errors
- Image errors
- Volume errors

---

## Step 3 - View Logs

```bash
kubectl logs POD_NAME
```

For multi-container Pods:

```bash
kubectl logs POD_NAME -c CONTAINER_NAME
```

---

## Step 4 - Execute Commands Inside the Pod

```bash
kubectl exec -it POD_NAME -- /bin/bash
```

or

```bash
kubectl exec -it POD_NAME -- /bin/sh
```

Useful for:

- Checking files
- Testing network connectivity
- Verifying environment variables

---

# Common Errors

## CrashLoopBackOff

Reasons:

- Application crash
- Invalid configuration
- Missing dependencies

---

## ImagePullBackOff

Reasons:

- Incorrect image name
- Image does not exist
- Private registry authentication failure

---

## Pending

Reasons:

- Insufficient CPU or Memory
- No suitable node
- PVC not available
- Taints and Tolerations

---

## CreateContainerConfigError

Reasons:

- Missing ConfigMap
- Missing Secret
- Invalid configuration

---

# 24. Hands-on Labs

## Lab 1 - Create an Nginx Pod

```bash
kubectl run nginx --image=nginx
```

Verify:

```bash
kubectl get pods
```

---

## Lab 2 - Describe the Pod

```bash
kubectl describe pod nginx
```

Observe:

- Events
- Node
- IP Address
- Container Status

---

## Lab 3 - View Logs

```bash
kubectl logs nginx
```

---

## Lab 4 - Delete the Pod

```bash
kubectl delete pod nginx
```

---

## Lab 5 - Create Pod Using YAML

```bash
kubectl apply -f pod.yaml
```

---

## Lab 6 - Export Existing YAML

```bash
kubectl get pod nginx -o yaml
```

Study every field.

---

# 25. Production Best Practices

✅ One application per Pod whenever possible

✅ Use meaningful labels

✅ Use declarative YAML instead of imperative commands

✅ Store manifests in Git

✅ Avoid editing live Pods directly

✅ Monitor Pod health continuously

✅ Use resource requests and limits (covered in later chapters)

✅ Keep container images lightweight

---

# 26. CKA Exam Tips

✔ Know how to create Pods quickly.

✔ Practice both imperative and declarative methods.

✔ Understand Pod lifecycle.

✔ Remember important troubleshooting commands.

✔ Be comfortable reading YAML.

✔ Use `kubectl explain` to understand resource fields.

---

# 27. Interview Questions

## Basic

1. What is a Pod?
2. Why is a Pod the smallest deployable unit?
3. Can a Pod contain multiple containers?
4. What is the Pause Container?
5. What is the purpose of labels?

---

## Intermediate

1. Explain the Pod lifecycle.
2. Difference between Pod and Container.
3. Explain Pod networking.
4. What are Init Containers?
5. What are Static Pods?

---

## Advanced

1. Explain how kubelet manages Pods.
2. Why does Kubernetes use Pods instead of containers directly?
3. How does Pod-to-Pod communication work?
4. Explain CrashLoopBackOff troubleshooting.
5. How do Static Pods differ from regular Pods?

---

## Scenario-Based

### Scenario 1

Your Pod is stuck in `Pending`.

How would you troubleshoot it?

---

### Scenario 2

The application inside a Pod keeps restarting.

What commands would you use to identify the root cause?

---

### Scenario 3

A Pod cannot pull its container image.

What are the possible reasons?

---

### Scenario 4

A developer accidentally deleted a Pod created with `kubectl run`.

Will Kubernetes recreate it? Why or why not?

---

### Scenario 5

How would you investigate a `CrashLoopBackOff` issue in a production environment?

---

# 28. Quick Cheat Sheet

Create Pod

```bash
kubectl run nginx --image=nginx
```

Apply YAML

```bash
kubectl apply -f pod.yaml
```

Get Pods

```bash
kubectl get pods
```

Describe Pod

```bash
kubectl describe pod POD_NAME
```

View Logs

```bash
kubectl logs POD_NAME
```

Execute Inside Pod

```bash
kubectl exec -it POD_NAME -- /bin/bash
```

Delete Pod

```bash
kubectl delete pod POD_NAME
```

Explain Resource

```bash
kubectl explain pod
```

---

# 29. Key Takeaways

- A Pod is the smallest deployable unit in Kubernetes.
- Pods can contain one or multiple containers.
- Every Pod has its own IP address.
- The Pause Container provides shared namespaces.
- Init Containers prepare the environment before the application starts.
- Static Pods are managed directly by kubelet.
- Troubleshooting starts with `kubectl get`, `describe`, and `logs`.
- Declarative YAML is the preferred approach in production.

---

# 30. Chapter Summary

Congratulations! 🎉

You have completed one of the most important Kubernetes topics.

In this chapter, you learned:

- Why Pods exist
- Pod architecture
- Pod lifecycle and phases
- Single-container and multi-container Pods
- Pause Container
- Pod networking
- Labels and Selectors
- Imperative and Declarative Pod creation
- YAML fundamentals
- Restart Policies
- Init Containers
- Static Pods
- Pod troubleshooting
- Production best practices
- CKA tips
- Interview preparation

With a strong understanding of Pods, you're now ready to learn how Kubernetes provides **high availability and scaling** using **ReplicaSets and Deployments**.

