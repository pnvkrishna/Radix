# K8S-13 - DaemonSets (Part 1)

> **Class Date:** 05-Jul-2026
>
> **Module:** DaemonSets


---

# 📚 Table of Contents

1. Introduction
2. Why DaemonSets?
3. What is a DaemonSet?
4. DaemonSet Architecture
5. DaemonSet Scheduling
6. DaemonSet vs Deployment vs StatefulSet
7. Common Production Use Cases
8. First DaemonSet YAML
9. Part 1 Summary

---

# 1. Introduction

Most Kubernetes workloads are application-focused.

Examples:

- Frontend
- Backend API
- Database
- Authentication Service

These workloads don't need to run on every node.

However, some workloads provide **cluster-wide infrastructure services**.

Examples:

- Log collection
- Metrics collection
- Network plugins
- Storage plugins
- Security monitoring

These workloads should automatically run on every suitable node.

---

# 2. Why DaemonSets?

Imagine a cluster:

```
Node-1

Node-2

Node-3
```

Each node generates:

- System logs
- Container logs
- CPU metrics
- Memory metrics
- Disk metrics

If only one logging Pod exists:

```
Node-1

↓

Logging Agent

↓

Can collect only accessible logs
```

Logs from other nodes would not be collected locally.

A better approach:

```
Node-1 → Agent

Node-2 → Agent

Node-3 → Agent
```

One agent runs on every node.

---

# 3. What is a DaemonSet?

A **DaemonSet** is a Kubernetes workload API that ensures **one Pod runs on every eligible node**.

As nodes are added or removed:

- New nodes receive a DaemonSet Pod automatically.
- Removed nodes have their DaemonSet Pods removed.

---

# Characteristics

✔ One Pod per eligible node

✔ Automatic scheduling

✔ Automatic scaling with cluster size

✔ Ideal for infrastructure components

---

# Example

Cluster:

```
3 Nodes
```

DaemonSet:

```
Node-1

↓

Fluent Bit

----------------

Node-2

↓

Fluent Bit

----------------

Node-3

↓

Fluent Bit
```

---

# 4. DaemonSet Architecture

```
             DaemonSet

                 │

     ┌───────────┼───────────┐

     ▼           ▼           ▼

  Node-1      Node-2      Node-3

     │           │           │

     ▼           ▼           ▼

 FluentBit   FluentBit   FluentBit
```

Each node runs exactly one DaemonSet Pod (unless node selectors, taints, or affinity rules change the eligible node set).

---

# 5. DaemonSet Scheduling

Unlike Deployments,

DaemonSets do **not** create a fixed number of replicas.

Instead:

```
Eligible Nodes

↓

One Pod Per Node
```

---

## Node Added

Before:

```
3 Nodes

↓

3 Pods
```

After adding a node:

```
4 Nodes

↓

4 Pods
```

The new node automatically receives a DaemonSet Pod.

---

## Node Removed

Before:

```
4 Nodes

↓

4 Pods
```

After removal:

```
3 Nodes

↓

3 Pods
```

The Pod on the removed node disappears.

---

# Scheduling Flow

```
DaemonSet Created

↓

Scheduler Places Pods

↓

One Pod Per Eligible Node

↓

Cluster Changes

↓

Pods Automatically Adjust
```

---

# 6. DaemonSet vs Deployment vs StatefulSet

| Feature | Deployment | StatefulSet | DaemonSet |
|----------|------------|-------------|-----------|
| Replica Count | Fixed | Fixed | One per eligible node |
| Stable Pod Names | ❌ | ✅ | ❌ |
| Persistent Storage | Optional | Usually Required | Rarely Needed |
| Infrastructure Components | ❌ | ❌ | ✅ |
| Databases | ❌ | ✅ | ❌ |
| Logging Agents | ❌ | ❌ | ✅ |

---

# 7. Common Production Use Cases

## Logging Agents

Examples:

- Fluent Bit
- Fluentd

```
Node

↓

Container Logs

↓

Fluent Bit

↓

Elasticsearch / OpenSearch / Loki
```

---

## Monitoring

Examples:

- Prometheus Node Exporter
- Node-level metrics collectors

Each node exports:

- CPU
- Memory
- Disk
- Filesystem
- Network

---

## Networking

Examples:

- Calico Node
- Cilium Agent

These components run on every node to provide networking functionality.

---

## Storage

Examples:

- CSI Node Plugins

These interact with local storage or attach volumes to Pods on each node.

---

## Security

Examples:

- Runtime security agents
- Vulnerability monitoring agents
- Compliance agents

Each node is monitored independently.

---

# 8. First DaemonSet YAML

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: fluent-bit

spec:

  selector:
    matchLabels:
      app: fluent-bit

  template:

    metadata:
      labels:
        app: fluent-bit

    spec:

      containers:

      - name: fluent-bit

        image: fluent/fluent-bit:3.1
```

---

## YAML Explained

### selector

```yaml
selector:
  matchLabels:
    app: fluent-bit
```

Matches Pods managed by the DaemonSet.

---

### template

Defines the Pod specification.

---

### containers

Specifies the container(s) that will run on every eligible node.

---

## Verify

Create:

```bash
kubectl apply -f daemonset.yaml
```

List DaemonSets:

```bash
kubectl get daemonset
```

List Pods:

```bash
kubectl get pods -o wide
```

You should see one Pod scheduled on each eligible node.

---

# 9. Part 1 Summary

In this section, you learned:

- Why DaemonSets exist
- One Pod per eligible node
- Automatic scheduling
- Cluster growth behavior
- DaemonSet architecture
- Differences from Deployments and StatefulSets
- Common infrastructure use cases
- Basic DaemonSet YAML

You now understand how Kubernetes deploys cluster-wide infrastructure components.

# K8S-13 - DaemonSets (Part 2)

> Class Date: 05-Jul-2026

---

# 📚 Table of Contents

10. Node Selectors
11. Node Affinity
12. Taints & Tolerations
13. Running on Control Plane Nodes
14. Rolling Updates
15. Update Strategies
16. Resource Management
17. Security Best Practices
18. Production Examples
19. Complete Production YAML
20. Summary

---

# 10. Node Selectors

By default,

DaemonSets run on every eligible node.

Sometimes,

you only want specific nodes.

Example

```
GPU Nodes

↓

GPU Monitoring Agent

-------------------------

Storage Nodes

↓

Storage Agent
```

NodeSelector allows this.

---

## Label Node

```
kubectl label node worker-1 disk=ssd
```

---

## YAML

```yaml
spec:

  template:

    spec:

      nodeSelector:

        disk: ssd
```

Only nodes having

```
disk=ssd
```

receive the Pod.

---

# Architecture

```
Cluster

├── worker-1 (disk=ssd)

├── worker-2 (disk=hdd)

├── worker-3 (disk=ssd)

↓

DaemonSet

↓

worker-1 ✅

worker-2 ❌

worker-3 ✅
```

---

# 11. Node Affinity

Node Affinity is more flexible than NodeSelector.

Example

```yaml
affinity:

  nodeAffinity:

    requiredDuringSchedulingIgnoredDuringExecution:

      nodeSelectorTerms:

      - matchExpressions:

        - key: disk

          operator: In

          values:

          - ssd
```

---

## Advantages

Supports

- In
- NotIn
- Exists
- DoesNotExist
- Gt
- Lt

Very useful in production.

---

# 12. Taints & Tolerations

Some nodes are protected.

Example

```
Database Node

↓

No Normal Workloads
```

Kubernetes uses **taints**.

Example

```
kubectl taint nodes worker-1 database=true:NoSchedule
```

Now nothing can schedule there.

---

## Toleration

DaemonSet

```yaml
tolerations:

- key: database

  operator: Equal

  value: "true"

  effect: NoSchedule
```

Now DaemonSet Pods may run there.

---

# Why?

Infrastructure agents often need to run everywhere,

including protected nodes.

Examples

- CNI
- CSI
- Logging
- Monitoring

---

# 13. Running on Control Plane Nodes

Modern Kubernetes control plane nodes are usually tainted.

Without tolerations,

DaemonSet Pods won't run there.

Production examples that commonly run on control plane nodes include:

- CNI node agents
- CSI node plugins
- Some monitoring agents

---

Example

```
Control Plane

↓

Taint

↓

DaemonSet

↓

Toleration

↓

Scheduled
```

---

# 14. Rolling Updates

DaemonSets also support rolling updates.

Suppose

```
Fluent Bit v3.1
```

needs upgrading to

```
v3.2
```

Update order

```
Node-1

↓

Updated

↓

Ready

↓

Node-2

↓

Updated

↓

Ready

↓

Node-3
```

One node at a time.

---

Benefits

✔ Lower risk

✔ Continuous logging

✔ Safer upgrades

---

# 15. Update Strategy

Default

```yaml
updateStrategy:

  type: RollingUpdate
```

---

Another option

```
OnDelete
```

Example

```yaml
updateStrategy:

  type: OnDelete
```

Pods update only after manual deletion.

Useful when administrators need precise control over update timing.

---

# 16. Resource Management

Infrastructure Pods should have resource requests.

Example

```yaml
resources:

  requests:

    cpu: 100m

    memory: 128Mi

  limits:

    cpu: 500m

    memory: 512Mi
```

Without limits,

agents can consume excessive resources.

---

# 17. Security Best Practices

Use

```
readOnlyRootFilesystem
```

where supported.

---

Avoid

```
privileged: true
```

unless absolutely necessary.

Some system agents require elevated permissions, but grant only the minimum required privileges.

---

Run as non-root when possible.

---

Use read-only mounts whenever practical.

---

Avoid mounting the host filesystem broadly.

Mount only required directories.

---

# 18. Production Examples

---

## Fluent Bit

```
Node

↓

Container Logs

↓

Fluent Bit

↓

OpenSearch
```

Runs on every node.

---

## Prometheus Node Exporter

```
Node

↓

CPU

Memory

Filesystem

↓

Prometheus
```

One exporter per node.

---

## Calico

```
Every Node

↓

Calico Agent

↓

Networking
```

---

## CSI Node Plugin

```
Every Node

↓

CSI

↓

Volume Mounts
```

---

# 19. Complete Production YAML

```yaml
apiVersion: apps/v1

kind: DaemonSet

metadata:

  name: fluent-bit

spec:

  selector:

    matchLabels:

      app: fluent-bit

  updateStrategy:

    type: RollingUpdate

  template:

    metadata:

      labels:

        app: fluent-bit

    spec:

      tolerations:

      - operator: Exists

      containers:

      - name: fluent-bit

        image: fluent/fluent-bit:3.1

        resources:

          requests:

            cpu: 100m

            memory: 128Mi

          limits:

            cpu: 500m

            memory: 512Mi
```

---

Explanation

### updateStrategy

Controls updates.

---

### tolerations

Allows scheduling onto tainted nodes where appropriate.

---

### resources

Protects cluster resources.

---

# 20. Part 2 Summary

You learned

- NodeSelector
- Node Affinity
- Taints
- Tolerations
- Control Plane scheduling
- Rolling Updates
- Update Strategies
- Resource Management
- Security Best Practices
- Production DaemonSets

You now understand how production clusters deploy infrastructure software across Kubernetes nodes safely and efficiently.

# K8S-13 - DaemonSets (Part 3)

> **Class Date:** 05-Jul-2026

---

# 📚 Table of Contents

21. DaemonSet Lifecycle
22. Cluster Scaling Behavior
23. Failure Recovery
24. Common Mistakes
25. Troubleshooting
26. Production Best Practices
27. Hands-on Labs
28. CKA Exam Tips
29. Interview Questions
30. DaemonSet Cheat Sheet
31. Chapter Summary

---

# 21. DaemonSet Lifecycle

A DaemonSet continuously watches the cluster for eligible nodes.

Lifecycle:

```
DaemonSet Created

↓

Scheduler Evaluates Eligible Nodes

↓

One Pod Per Eligible Node

↓

New Node Added

↓

New DaemonSet Pod Created

↓

Node Removed

↓

DaemonSet Pod Removed
```

Unlike Deployments, you don't scale a DaemonSet by increasing replicas.

The number of Pods is determined by the number of eligible nodes.

---

# 22. Cluster Scaling Behavior

## Scale Out

Initial cluster:

```
worker-1
worker-2
worker-3
```

DaemonSet:

```
3 Pods
```

A new node joins:

```
worker-4
```

Result:

```
worker-1 → fluent-bit

worker-2 → fluent-bit

worker-3 → fluent-bit

worker-4 → fluent-bit
```

No manual action is required.

---

## Scale In

Before:

```
4 Nodes

↓

4 DaemonSet Pods
```

Node removed:

```
worker-4
```

Result:

```
3 Nodes

↓

3 DaemonSet Pods
```

The Pod on the removed node disappears automatically.

---

# 23. Failure Recovery

## Pod Failure

Suppose:

```
fluent-bit Pod
```

crashes on:

```
worker-2
```

DaemonSet controller detects it.

```
worker-2

↓

Old Pod Removed

↓

New Pod Created

↓

Logging Restored
```

---

## Node Failure

```
worker-2

↓

Offline
```

The DaemonSet Pod is unavailable because the node is unavailable.

When the node returns, or when a replacement node joins the cluster, the DaemonSet controller ensures the desired Pod is running on eligible nodes.

---

## Node Drain

Example:

```bash
kubectl drain worker-2 --ignore-daemonsets
```

Why?

DaemonSet Pods are normally **ignored during drain** because they are managed separately by the DaemonSet controller.

After maintenance:

```bash
kubectl uncordon worker-2
```

The DaemonSet reconciles the desired state on that node.

---

# 24. Common Mistakes

## Using DaemonSet for Web Applications

❌ Incorrect

```
NGINX

↓

DaemonSet
```

Every node gets a copy unnecessarily.

Use a Deployment instead.

---

## Forgetting Resource Limits

Infrastructure Pods run continuously.

Without limits:

```
Logging Agent

↓

High Memory Usage

↓

Node Pressure
```

Always define resource requests and limits.

---

## Missing Tolerations

Protected nodes:

```
Control Plane

↓

Taint

↓

No DaemonSet Pod
```

If the infrastructure agent must run there, configure the required tolerations.

---

## Overusing Privileged Mode

Many agents **do not** require full privileged access.

Grant only the permissions required for the workload.

---

## Mounting the Entire Host Filesystem

Avoid:

```yaml
hostPath:
  path: /
```

Prefer mounting only the directories that are actually needed, such as specific log directories.

---

# 25. Troubleshooting

## Check DaemonSet

```bash
kubectl get daemonset
```

---

## Describe DaemonSet

```bash
kubectl describe daemonset fluent-bit
```

Review:

- Desired Pods
- Current Pods
- Ready Pods
- Events

---

## Check Pods

```bash
kubectl get pods -o wide
```

Verify there is one Pod on each eligible node.

---

## Check Nodes

```bash
kubectl get nodes
```

Confirm node status:

```
Ready
```

---

## Check Labels

```bash
kubectl get nodes --show-labels
```

Useful when using:

- `nodeSelector`
- Node affinity

---

## Check Taints

```bash
kubectl describe node worker-1
```

Review:

```
Taints:
```

Verify that your DaemonSet has matching tolerations if needed.

---

## Check Logs

```bash
kubectl logs daemonset/fluent-bit
```

Or inspect a specific Pod:

```bash
kubectl logs fluent-bit-xxxxx
```

---

# Common Problems

| Problem | Cause | Solution |
|---------|-------|----------|
| Missing Pod on a node | Node selector or affinity mismatch | Verify labels and scheduling rules |
| Pending Pod | Taints or insufficient resources | Check events and tolerations |
| CrashLoopBackOff | Configuration or application issue | Review Pod logs |
| No Pods on control plane | Missing toleration | Add required toleration |
| High resource usage | No limits configured | Set CPU and memory requests/limits |

---

# 26. Production Best Practices

## Keep Images Updated

Use stable, supported image versions.

Avoid floating tags such as `latest` in production.

---

## Set Resource Requests & Limits

Prevent infrastructure agents from starving application workloads.

---

## Use Rolling Updates

Allow gradual upgrades with minimal operational risk.

---

## Monitor Agent Health

Track:

- Pod restarts
- CPU usage
- Memory usage
- Log forwarding status
- Node coverage

---

## Secure Host Access

Grant only the host mounts and Linux capabilities that are required.

Avoid excessive permissions.

---

## Test Node Replacement

Regularly verify that:

- New nodes automatically receive DaemonSet Pods.
- Logging and monitoring resume correctly.

---

# 27. Hands-on Labs

## Lab 1

Create a DaemonSet.

Verify:

```bash
kubectl get daemonset
kubectl get pods -o wide
```

---

## Lab 2

Add a node to the cluster.

Observe that a new DaemonSet Pod is scheduled automatically.

---

## Lab 3

Label one node:

```bash
kubectl label node worker-1 disk=ssd
```

Update the DaemonSet to use:

```yaml
nodeSelector:
  disk: ssd
```

Verify scheduling.

---

## Lab 4

Add a taint:

```bash
kubectl taint nodes worker-1 logging=true:NoSchedule
```

Add the corresponding toleration to the DaemonSet.

Verify scheduling.

---

## Lab 5

Upgrade the container image.

Watch the rollout:

```bash
kubectl rollout status daemonset/fluent-bit
```

---

# 28. CKA Exam Tips

✔ Know when to use:

- Deployment
- StatefulSet
- DaemonSet

✔ Memorize:

- One Pod per eligible node
- No replica count
- Automatic scheduling

✔ Practice:

```bash
kubectl get daemonset
kubectl describe daemonset
kubectl rollout status daemonset/<name>
```

✔ Understand:

- Node selectors
- Affinity
- Taints
- Tolerations

---

# 29. Interview Questions

## Beginner

1. What is a DaemonSet?
2. How is a DaemonSet different from a Deployment?
3. Why are DaemonSets used for logging agents?
4. What happens when a new node joins the cluster?
5. Can a DaemonSet have replicas?

---

## Intermediate

1. Explain DaemonSet scheduling.
2. What are common DaemonSet workloads?
3. How do taints and tolerations affect DaemonSets?
4. How are DaemonSets updated?
5. How would you restrict a DaemonSet to GPU nodes?

---

## Advanced

1. Design a logging architecture using DaemonSets.
2. Explain how CSI node plugins use DaemonSets.
3. How would you troubleshoot missing DaemonSet Pods?
4. Compare DaemonSets with static Pods.
5. Explain how DaemonSets behave during cluster autoscaling.

---

## Scenario-Based

### Scenario 1

A new worker node joins the cluster.

What should happen automatically?

---

### Scenario 2

A DaemonSet Pod is missing from one node.

How would you troubleshoot it?

---

### Scenario 3

The logging agent consumes excessive memory.

What changes would you make?

---

### Scenario 4

A monitoring agent must also run on control plane nodes.

What Kubernetes feature enables this?

---

### Scenario 5

A team proposes using a DaemonSet for a REST API.

Would you recommend it? Why or why not?

---

# 30. DaemonSet Cheat Sheet

List DaemonSets:

```bash
kubectl get daemonset
```

Describe:

```bash
kubectl describe daemonset fluent-bit
```

Watch rollout:

```bash
kubectl rollout status daemonset/fluent-bit
```

List Pods:

```bash
kubectl get pods -o wide
```

Show node labels:

```bash
kubectl get nodes --show-labels
```

Drain a node:

```bash
kubectl drain worker-1 --ignore-daemonsets
```

---

# 31. Key Takeaways

- DaemonSets run one Pod on every eligible node.
- They automatically adapt to cluster growth and shrinkage.
- They are ideal for infrastructure services such as logging, monitoring, networking, and storage agents.
- Node selectors, affinity, taints, and tolerations control placement.
- Use resource limits and least-privilege security settings.
- Rolling updates provide safe upgrades.
- DaemonSets are not a replacement for Deployments.

---

# 32. Chapter Summary

Congratulations! 🎉

You have completed **K8S-13 – DaemonSets**.

In this chapter, you learned:

- Why DaemonSets exist
- One Pod per eligible node
- Scheduling behavior
- Node selectors
- Node affinity
- Taints and tolerations
- Control plane scheduling
- Rolling updates
- Resource management
- Security best practices
- Failure recovery
- Troubleshooting
- Hands-on labs
- CKA preparation
- Interview questions

You now understand how Kubernetes deploys and manages cluster-wide infrastructure services consistently across every eligible node.