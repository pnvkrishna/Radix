# K8S-19 - Scheduling (NodeSelector, Node Affinity, Pod Affinity, Anti-Affinity, Taints & Tolerations) (Part 1)

> **Class Date:** 11-Jul-2026
>
> **Module:** Scheduling


---

# 📚 Table of Contents

1. Introduction
2. Kubernetes Scheduler
3. Scheduling Workflow
4. Node Labels
5. NodeSelector
6. NodeSelector YAML
7. Scheduling Failure
8. Part 1 Summary

---

# 1. Introduction

When a Pod is created, Kubernetes must decide:

```
Which Node should run this Pod?
```

This decision is made by the **kube-scheduler**.

Example:

```
Cluster

├── worker-1
├── worker-2
├── worker-3

↓

New Pod

↓

Scheduler Chooses

↓

worker-2
```

---

# 2. Kubernetes Scheduler

The **kube-scheduler** watches for Pods that do not yet have a node assigned.

Its job is to:

- Find candidate nodes
- Filter unsuitable nodes
- Score the remaining nodes
- Bind the Pod to the best node

---

## Scheduler Decision Flow

```
New Pod

↓

Pending

↓

Scheduler

↓

Filter Nodes

↓

Score Nodes

↓

Select Best Node

↓

Bind Pod
```

---

# 3. Scheduling Workflow

The scheduler considers many factors, including:

- Resource requests (CPU and memory)
- Node readiness
- Node labels
- Taints and tolerations
- Affinity and anti-affinity rules
- Volume constraints
- Topology constraints

---

## Simplified Flow

```
Pod Created

↓

Pending

↓

Filter

↓

Score

↓

Best Node

↓

Running
```

---

# 4. Node Labels

Nodes contain labels that describe their characteristics.

Show labels:

```bash
kubectl get nodes --show-labels
```

Example:

```
worker-1

gpu=true

zone=ap-south-1a

disk=ssd
```

Custom label:

```bash
kubectl label node worker-1 gpu=true
```

Remove a label:

```bash
kubectl label node worker-1 gpu-
```

---

# 5. NodeSelector

`nodeSelector` is the simplest scheduling constraint.

It matches Pods to nodes with specific labels.

Example:

```yaml
spec:
  nodeSelector:
    gpu: "true"
```

---

## Architecture

```
Nodes

worker-1

gpu=true

------------

worker-2

gpu=false

↓

Pod

↓

nodeSelector

↓

worker-1
```

---

## Matching Rule

The Pod is scheduled **only** to nodes where all specified labels match.

Example:

```yaml
nodeSelector:
  gpu: "true"
  disk: "ssd"
```

This requires:

```
gpu=true

AND

disk=ssd
```

---

# 6. NodeSelector YAML

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: gpu-app

spec:

  nodeSelector:
    gpu: "true"

  containers:
  - name: app
    image: nginx:1.27
```

---

# YAML Explained

## nodeSelector

Requires the target node to have:

```
gpu=true
```

Otherwise, the Pod cannot be scheduled there.

---

## Verify

Create:

```bash
kubectl apply -f pod.yaml
```

Describe:

```bash
kubectl describe pod gpu-app
```

Look for:

```
Node:
Events:
```

---

# 7. Scheduling Failure

Suppose:

```
worker-1

gpu=false

------------

worker-2

gpu=false
```

Pod:

```yaml
nodeSelector:
  gpu: "true"
```

Result:

```
No Matching Node

↓

Pod Remains Pending
```

---

## Typical Event

```text
0/2 nodes are available:
2 node(s) didn't match Pod's node selector.
```

Use:

```bash
kubectl describe pod gpu-app
```

to inspect scheduling events.

---

# Production Example

```
GPU Workload

↓

nodeSelector

↓

gpu=true

↓

GPU Node
```

Similarly:

- Storage-intensive workloads → `disk=ssd`
- Compliance workloads → `zone=secure`
- ARM workloads → `kubernetes.io/arch=arm64`

---

# 8. Part 1 Summary

You learned:

- Kubernetes scheduler
- Scheduling workflow
- Node labels
- `nodeSelector`
- Scheduling failures
- Basic scheduling YAML

You now understand how Kubernetes uses node labels and `nodeSelector` to place Pods on appropriate nodes.

# K8S-19 - Scheduling (Part 2)

> **Class Date:** 11-Jul-2026

---

# 📚 Table of Contents

9. Node Affinity
10. Required Node Affinity
11. Preferred Node Affinity
12. matchExpressions
13. Pod Affinity
14. Pod Anti-Affinity
15. Topology Domains
16. Production YAML
17. Part 2 Summary

---

# 9. Node Affinity

Node Affinity is a more expressive version of `nodeSelector`.

Instead of matching only exact label values, it supports:

- Multiple operators
- Preferred rules
- Required rules
- Flexible matching

---

## nodeSelector vs Node Affinity

| Feature | nodeSelector | Node Affinity |
|----------|--------------|---------------|
| Exact Match | ✅ | ✅ |
| Preferred Rules | ❌ | ✅ |
| Required Rules | ❌ | ✅ |
| Set-based Operators | ❌ | ✅ |
| Flexible Scheduling | ❌ | ✅ |

---

# 10. Required Node Affinity

This is a **hard requirement**.

If no node satisfies the rule:

```
Pod

↓

Pending
```

Example:

```yaml
affinity:

  nodeAffinity:

    requiredDuringSchedulingIgnoredDuringExecution:

      nodeSelectorTerms:

      - matchExpressions:

        - key: gpu

          operator: In

          values:

          - "true"
```

---

## Architecture

```
Nodes

↓

gpu=true

↓

Required Affinity

↓

Pod Scheduled
```

If no node has:

```
gpu=true
```

the Pod remains **Pending**.

---

# 11. Preferred Node Affinity

This is a **soft preference**.

Scheduler behavior:

```
Preferred Node Available

↓

Use It

----------------

Not Available

↓

Choose Another Suitable Node
```

Example:

```yaml
preferredDuringSchedulingIgnoredDuringExecution:

- weight: 100

  preference:

    matchExpressions:

    - key: disk

      operator: In

      values:

      - ssd
```

---

## Why Use Preferred Rules?

Examples:

- Prefer SSD storage
- Prefer GPU nodes
- Prefer specific availability zones

Without preventing scheduling if the preferred option is unavailable.

---

# 12. matchExpressions

Affinity rules use `matchExpressions`.

Supported operators:

| Operator | Meaning |
|----------|---------|
| In | Value must match |
| NotIn | Value must not match |
| Exists | Label key exists |
| DoesNotExist | Label key absent |
| Gt | Greater than |
| Lt | Less than |

---

## Example

```yaml
matchExpressions:

- key: zone

  operator: In

  values:

  - ap-south-1a

  - ap-south-1b
```

Meaning:

```
Zone A

OR

Zone B
```

---

## Multiple Expressions

```yaml
matchExpressions:

- key: gpu

  operator: In

  values:

  - "true"

- key: disk

  operator: In

  values:

  - ssd
```

Result:

```
gpu=true

AND

disk=ssd
```

---

# 13. Pod Affinity

Pod Affinity places Pods **close together**.

Example:

```
Frontend

↓

Backend

↓

Same Node
```

Use cases:

- Reduce network latency
- Improve communication speed
- Cache locality

---

## Example

```yaml
podAffinity:

  requiredDuringSchedulingIgnoredDuringExecution:

  - labelSelector:

      matchLabels:

        app: backend

    topologyKey: kubernetes.io/hostname
```

Meaning:

Schedule this Pod onto a node that is already running a Pod labeled:

```
app=backend
```

---

# 14. Pod Anti-Affinity

Pod Anti-Affinity spreads Pods apart.

Example:

```
Node-1

Frontend

------------

Node-2

Frontend

------------

Node-3

Frontend
```

Instead of:

```
Node-1

Frontend

Frontend

Frontend
```

---

## Example

```yaml
podAntiAffinity:

  requiredDuringSchedulingIgnoredDuringExecution:

  - labelSelector:

      matchLabels:

        app: frontend

    topologyKey: kubernetes.io/hostname
```

---

## Benefits

- High Availability
- Fault Tolerance
- Better Resource Distribution

---

# 15. Topology Domains

The `topologyKey` determines **where** affinity or anti-affinity applies.

Common keys:

| topologyKey | Meaning |
|-------------|---------|
| `kubernetes.io/hostname` | Same or different node |
| `topology.kubernetes.io/zone` | Same or different availability zone |
| `topology.kubernetes.io/region` | Same or different region |

---

## Example

```
Region

↓

Zone

↓

Node
```

Anti-affinity across zones improves resilience against zone failures.

---

# 16. Production YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: payment

spec:

  replicas: 3

  selector:

    matchLabels:

      app: payment

  template:

    metadata:

      labels:

        app: payment

    spec:

      affinity:

        nodeAffinity:

          requiredDuringSchedulingIgnoredDuringExecution:

            nodeSelectorTerms:

            - matchExpressions:

              - key: disk

                operator: In

                values:

                - ssd

        podAntiAffinity:

          preferredDuringSchedulingIgnoredDuringExecution:

          - weight: 100

            podAffinityTerm:

              labelSelector:

                matchLabels:

                  app: payment

              topologyKey: kubernetes.io/hostname

      containers:

      - name: payment

        image: payment:v2
```

---

# YAML Explained

## Required Node Affinity

Only schedule on nodes with:

```
disk=ssd
```

---

## Preferred Pod Anti-Affinity

Prefer spreading replicas across different nodes.

If that isn't possible, scheduling can still proceed.

---

# Production Example

```
Three Replicas

↓

Node-1

payment

------------

Node-2

payment

------------

Node-3

payment
```

This improves resilience against single-node failures.

---

# 17. Part 2 Summary

You learned:

- Node Affinity
- Required affinity
- Preferred affinity
- `matchExpressions`
- Pod Affinity
- Pod Anti-Affinity
- Topology domains
- Production scheduling YAML

You now understand how Kubernetes schedules Pods based on node characteristics and workload placement preferences.

# K8S-19 - Scheduling (Part 3)

> **Class Date:** 11-Jul-2026

---

# 📚 Table of Contents

18. Taints
19. Tolerations
20. Taint Effects
21. Combining Affinity & Tolerations
22. Scheduler Decision Flow
23. Troubleshooting Pending Pods
24. Production Best Practices
25. Hands-on Labs
26. CKA Exam Tips
27. Interview Questions
28. Scheduling Cheat Sheet
29. Chapter Summary

---

# 18. Taints

A taint is applied to a **Node**.

Purpose:

Prevent Pods from running on that node unless they have a matching toleration.

---

## Architecture

```
Node

↓

gpu=true

↓

Taint Applied

↓

Reject Normal Pods
```

---

## Add a Taint

```bash
kubectl taint nodes worker-1 \
gpu=true:NoSchedule
```

Syntax:

```
key=value:effect
```

Example:

```
gpu=true:NoSchedule
```

---

## View Taints

```bash
kubectl describe node worker-1
```

Look for:

```
Taints:
```

---

## Remove a Taint

```bash
kubectl taint nodes worker-1 \
gpu=true:NoSchedule-
```

---

# 19. Tolerations

A toleration is configured on a **Pod**.

Example:

```yaml
tolerations:

- key: gpu

  operator: Equal

  value: "true"

  effect: NoSchedule
```

Meaning:

```
Node

↓

gpu=true

↓

NoSchedule

↓

Pod Tolerates

↓

Scheduling Allowed
```

Remember:

A toleration **permits** scheduling onto a tainted node; it does **not** require it.

---

# 20. Taint Effects

There are three taint effects.

---

## NoSchedule

Pods without a matching toleration are **not scheduled**.

```
Node

↓

NoSchedule

↓

Pod Rejected
```

---

## PreferNoSchedule

A soft preference.

The scheduler tries to avoid the node but may still use it if necessary.

---

## NoExecute

Existing Pods without a matching toleration are **evicted**.

Example:

```
Node

↓

NoExecute

↓

Pod Evicted
```

---

## Taint Effect Comparison

| Effect | New Pods | Existing Pods |
|---------|----------|---------------|
| NoSchedule | Blocked | Continue Running |
| PreferNoSchedule | Avoid if possible | Continue Running |
| NoExecute | Blocked | Evicted (unless tolerated) |

---

# 21. Combining Affinity & Tolerations

Production GPU example:

Node:

```
gpu=true

Taint:

gpu=true:NoSchedule
```

Pod:

```yaml
affinity:

  nodeAffinity:

    requiredDuringSchedulingIgnoredDuringExecution:

      nodeSelectorTerms:

      - matchExpressions:

        - key: gpu

          operator: In

          values:

          - "true"

tolerations:

- key: gpu

  operator: Equal

  value: "true"

  effect: NoSchedule
```

---

## Result

```
Node Matches Affinity

↓

Pod Tolerates Taint

↓

Pod Scheduled
```

This ensures that:

- Only GPU workloads target GPU nodes.
- General workloads cannot accidentally consume GPU resources.

---

# 22. Scheduler Decision Flow

A simplified scheduling process:

```
Pod Created

↓

Filter Nodes

↓

Check Resources

↓

Check Taints & Tolerations

↓

Check Affinity Rules

↓

Score Nodes

↓

Bind Pod
```

If no node satisfies all required constraints:

```
Pod

↓

Pending
```

---

# 23. Troubleshooting Pending Pods

## Describe the Pod

```bash
kubectl describe pod <pod-name>
```

Common events:

```
0/5 nodes are available.

2 node(s) had untolerated taint.

3 node(s) didn't match Pod affinity.
```

---

## Show Node Labels

```bash
kubectl get nodes --show-labels
```

---

## Show Taints

```bash
kubectl describe node <node-name>
```

---

## Check Resources

```bash
kubectl top nodes
```

or:

```bash
kubectl describe node <node-name>
```

Review:

- Allocatable resources
- Requested resources

---

# Common Scheduling Problems

| Problem | Possible Cause | Solution |
|----------|----------------|----------|
| Pod Pending | No matching node labels | Verify `nodeSelector` or affinity |
| Untolerated taint | Missing toleration | Add appropriate toleration if intended |
| No suitable resources | Insufficient CPU or memory | Increase capacity or reduce requests |
| Affinity conflict | Required rules too restrictive | Review affinity expressions |
| All replicas on one node | Missing anti-affinity or topology spread | Add scheduling constraints |

---

# 24. Production Best Practices

## Use Standard Node Labels

Examples:

```
topology.kubernetes.io/zone

kubernetes.io/arch

node.kubernetes.io/instance-type
```

Avoid unnecessary custom labels when standard labels already exist.

---

## Reserve Special Nodes

Examples:

- GPU nodes
- High-memory nodes
- Compliance-restricted nodes

Use:

- Taints
- Tolerations
- Node Affinity

together.

---

## Spread Critical Workloads

Prefer:

- Pod Anti-Affinity
- Topology Spread Constraints

to avoid placing all replicas on a single node.

> **Modern Note:** For evenly distributing replicas across nodes or zones, **Topology Spread Constraints** are often preferred over anti-affinity because they provide more predictable balancing. They are widely used in modern production clusters.

---

## Avoid Over-Constraining

Too many required scheduling rules can leave Pods permanently Pending.

Use preferred rules where appropriate.

---

# 25. Hands-on Labs

## Lab 1

Label a node:

```bash
kubectl label node worker-1 gpu=true
```

Deploy a Pod using:

```yaml
nodeSelector:
  gpu: "true"
```

---

## Lab 2

Apply a taint:

```bash
kubectl taint nodes worker-1 gpu=true:NoSchedule
```

Observe a normal Pod remain Pending.

---

## Lab 3

Add a toleration.

Verify the Pod schedules successfully.

---

## Lab 4

Deploy three replicas.

Add Pod Anti-Affinity.

Observe replica distribution across nodes.

---

## Lab 5

Create required Node Affinity.

Verify scheduling only occurs on matching nodes.

---

# 26. CKA Exam Tips

✔ Understand:

- NodeSelector
- Node Affinity
- Pod Affinity
- Pod Anti-Affinity
- Taints
- Tolerations

✔ Practice:

```bash
kubectl label node
kubectl taint nodes
kubectl describe pod
kubectl describe node
```

✔ Remember:

- Taints are applied to **Nodes**.
- Tolerations are applied to **Pods**.
- Tolerations allow scheduling; they do not guarantee it.

---

# 27. Interview Questions

## Beginner

1. What is `nodeSelector`?
2. What is Node Affinity?
3. What is a taint?
4. What is a toleration?
5. Why use Pod Anti-Affinity?

---

## Intermediate

1. Compare `nodeSelector` and Node Affinity.
2. Explain `NoSchedule`, `PreferNoSchedule`, and `NoExecute`.
3. How does Pod Affinity differ from Node Affinity?
4. Why would a Pod remain Pending?
5. How would you reserve GPU nodes?

---

## Advanced

1. Design scheduling for a multi-zone production cluster.
2. Explain how affinity, taints, and tolerations work together.
3. Compare Pod Anti-Affinity and Topology Spread Constraints.
4. Design scheduling for AI/ML workloads with dedicated GPU nodes.
5. Troubleshoot a Deployment where every Pod remains Pending.

---

## Scenario-Based

### Scenario 1

Your company has GPU nodes used only for ML workloads.

How would you prevent normal applications from using them?

---

### Scenario 2

All replicas of a Deployment are scheduled onto one node.

How would you improve availability?

---

### Scenario 3

A Pod has the correct toleration but still remains Pending.

What other scheduling rules would you investigate?

---

### Scenario 4

A node is placed into maintenance and receives a `NoExecute` taint.

What happens to existing Pods?

---

### Scenario 5

Your application should prefer SSD nodes but still run elsewhere if SSD nodes are unavailable.

Which scheduling feature would you choose?

---

# 28. Scheduling Cheat Sheet

Show node labels:

```bash
kubectl get nodes --show-labels
```

Add label:

```bash
kubectl label node worker-1 gpu=true
```

Remove label:

```bash
kubectl label node worker-1 gpu-
```

Add taint:

```bash
kubectl taint nodes worker-1 gpu=true:NoSchedule
```

Remove taint:

```bash
kubectl taint nodes worker-1 gpu=true:NoSchedule-
```

Describe Pod:

```bash
kubectl describe pod <pod-name>
```

Describe Node:

```bash
kubectl describe node <node-name>
```

---

# 29. Chapter Summary

Congratulations! 🎉

You have completed **K8S-19 – Scheduling**.

In this chapter, you learned:

- Kubernetes Scheduler
- Node labels
- `nodeSelector`
- Node Affinity
- Pod Affinity
- Pod Anti-Affinity
- Taints
- Tolerations
- Scheduler workflow
- Troubleshooting Pending Pods
- Production scheduling strategies
- Hands-on labs
- CKA preparation
- Interview questions

You now understand how Kubernetes places workloads efficiently while meeting resource, availability, and policy requirements.

