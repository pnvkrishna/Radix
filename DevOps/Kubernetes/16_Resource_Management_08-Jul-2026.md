# K8S-16 - Resource Management (Requests, Limits & QoS) (Part 1)

> **Class Date:** 08-Jul-2026
>
> **Module:** Resource Management


---

# 📚 Table of Contents

1. Introduction
2. Why Resource Management?
3. CPU vs Memory
4. What are Requests?
5. What are Limits?
6. Scheduler Behavior
7. First Resource YAML
8. Resource Units
9. Part 1 Summary

---

# 1. Introduction

Every Kubernetes node has **limited resources**.

Example:

```
Node

CPU : 8 Cores

Memory : 32 GiB
```

Multiple Pods compete for these resources.

Without proper resource management:

- One Pod may consume all CPU.
- One Pod may exhaust memory.
- Other applications may become slow or fail.
- The node may become unstable.

Resource requests and limits help Kubernetes manage this fairly.

---

# 2. Why Resource Management?

Imagine three Pods:

```
Node

CPU = 4

Memory = 8Gi

↓

Frontend

↓

Backend

↓

Database
```

If the backend suddenly consumes all memory:

```
Frontend

↓

Slow

Backend

↓

Memory Explosion

Database

↓

Killed
```

Production systems must prevent this.

---

# 3. CPU vs Memory

CPU and memory behave differently.

---

## CPU

CPU is **compressible**.

If an application wants more CPU than its limit, Kubernetes (through the Linux kernel's CPU control mechanisms) throttles CPU usage.

```
CPU Needed

↓

2 CPUs

↓

Limit

↓

1 CPU

↓

Runs More Slowly
```

The container keeps running but may perform more slowly.

---

## Memory

Memory is **not compressible**.

If an application exceeds its memory limit:

```
Memory Used

↓

Limit Exceeded

↓

OOM Kill

↓

Container Restart
```

The container is terminated because memory cannot be reclaimed in the same way as CPU.

---

# 4. What are Requests?

A **request** is the minimum amount of CPU or memory that Kubernetes reserves for a Pod when scheduling it.

Example:

```yaml
resources:

  requests:

    cpu: "500m"

    memory: "512Mi"
```

Meaning:

```
Scheduler

↓

Needs

0.5 CPU

512Mi Memory

↓

Find Suitable Node
```

---

## Important

Requests are primarily used for **scheduling**.

The scheduler ensures that a node has enough allocatable resources before placing the Pod there.

---

# 5. What are Limits?

A **limit** defines the maximum amount of CPU or memory a container is allowed to use.

Example:

```yaml
resources:

  limits:

    cpu: "1"

    memory: "1Gi"
```

Meaning:

```
Container

↓

Maximum

1 CPU

1Gi Memory
```

---

## CPU Limit

If the application tries to use more than the configured CPU limit:

```
CPU Requested

↓

2 CPUs

↓

Limit

↓

1 CPU

↓

CPU Throttled
```

---

## Memory Limit

If the application exceeds its memory limit:

```
Memory

↓

Exceeded

↓

OOMKilled
```

---

# 6. Scheduler Behavior

The Kubernetes scheduler considers **requests**, not limits, when deciding where to place Pods.

Example:

Node:

```
CPU = 4

Memory = 8Gi
```

Pod:

```yaml
requests:

  cpu: 2

  memory: 2Gi
```

Scheduler:

```
Node Has Capacity

↓

Schedule Pod
```

If the node lacks sufficient requested resources:

```
No Suitable Node

↓

Pod Remains Pending
```

---

# Scheduling Flow

```
Pod Created

↓

Read Requests

↓

Check Node Capacity

↓

Suitable Node Found

↓

Pod Scheduled
```

---

# 7. First Resource YAML

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: nginx

spec:

  containers:

  - name: nginx

    image: nginx:1.27

    resources:

      requests:

        cpu: "250m"

        memory: "256Mi"

      limits:

        cpu: "500m"

        memory: "512Mi"
```

---

## YAML Explained

### requests

Minimum resources reserved for scheduling.

---

### limits

Maximum resources the container may consume.

---

## Verify

Create:

```bash
kubectl apply -f pod.yaml
```

Describe:

```bash
kubectl describe pod nginx
```

Review:

```
Requests:

Limits:
```

---

# 8. Resource Units

## CPU

| Value | Meaning |
|-------|----------|
| `1000m` | 1 CPU |
| `500m` | 0.5 CPU |
| `250m` | 0.25 CPU |
| `100m` | 0.1 CPU |

Examples:

```yaml
cpu: "250m"
```

```yaml
cpu: "2"
```

---

## Memory

| Value | Meaning |
|-------|----------|
| `256Mi` | 256 Mebibytes |
| `512Mi` | 512 MiB |
| `1Gi` | 1024 MiB |
| `2Gi` | 2048 MiB |

Example:

```yaml
memory: "512Mi"
```

Prefer binary units (`Mi`, `Gi`) for clarity and consistency.

---

# Production Example

```
Frontend

Requests

250m CPU

256Mi RAM

------------

Backend

Requests

500m CPU

1Gi RAM

------------

Database

Requests

2 CPU

4Gi RAM
```

Each workload reserves the resources it needs to operate reliably.

---

# 9. Part 1 Summary

You learned:

- Why resource management matters
- CPU vs memory behavior
- Requests
- Limits
- Scheduler behavior
- Resource units
- Basic resource YAML

You now understand how Kubernetes reserves and constrains CPU and memory for workloads.

# K8S-16 - Resource Management (Part 2)

> **Class Date:** 08-Jul-2026

---

# 📚 Table of Contents

10. CPU Throttling
11. OOMKilled
12. Quality of Service (QoS)
13. ResourceQuota
14. LimitRange
15. Namespace Resource Governance
16. Resource Monitoring
17. Production YAML
18. Part 2 Summary

---

# 10. CPU Throttling

CPU is a **compressible** resource.

If a container attempts to use more CPU than its configured limit, Kubernetes (through Linux cgroups) throttles CPU usage instead of terminating the container.

Example:

```yaml
resources:
  requests:
    cpu: "500m"
  limits:
    cpu: "1"
```

Application demand:

```
Needs

2 CPU

↓

Limit

1 CPU

↓

CPU Throttled
```

---

## Symptoms

Applications may exhibit:

- Higher response times
- Increased request latency
- Slower batch processing
- Lower throughput

The container generally continues running.

---

## Production Recommendation

Avoid unnecessarily low CPU limits for CPU-intensive workloads.

Measure real usage before tuning.

---

# 11. OOMKilled

OOM = **Out Of Memory**

Memory cannot be throttled like CPU.

Example:

```yaml
resources:
  limits:
    memory: "512Mi"
```

Application usage:

```
512Mi

↓

700Mi

↓

OOMKilled
```

The Linux kernel terminates the container.

---

## Detect OOMKilled

```bash
kubectl describe pod <pod-name>
```

Example:

```
Last State:

Terminated

Reason: OOMKilled
```

---

## Verify Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

## Production Recommendation

Do not simply increase memory limits.

First determine:

- Memory leak?
- Larger workload?
- Incorrect sizing?
- Temporary spike?

Use monitoring data to guide changes.

---

# 12. Quality of Service (QoS)

Kubernetes assigns every Pod a QoS class.

QoS influences eviction priority during node resource pressure.

---

## Guaranteed

Every container has:

- CPU request = CPU limit
- Memory request = Memory limit

Example:

```yaml
resources:
  requests:
    cpu: "1"
    memory: "1Gi"

  limits:
    cpu: "1"
    memory: "1Gi"
```

QoS:

```
Guaranteed
```

Highest eviction priority (last to be evicted).

---

## Burstable

Requests and limits differ.

Example:

```yaml
requests:
  cpu: "500m"
  memory: "512Mi"

limits:
  cpu: "1"
  memory: "1Gi"
```

QoS:

```
Burstable
```

Most production applications use this class.

---

## BestEffort

No requests or limits defined.

Example:

```yaml
resources: {}
```

QoS:

```
BestEffort
```

Lowest protection during node pressure.

---

# QoS Comparison

| QoS Class | Requests | Limits | Eviction Priority |
|-----------|----------|--------|-------------------|
| Guaranteed | Required | Equal to requests | Last |
| Burstable | Defined | Optional or higher | Middle |
| BestEffort | None | None | First |

---

# 13. ResourceQuota

A **ResourceQuota** limits the total resources that can be consumed within a namespace.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota

metadata:
  name: production-quota

spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    pods: "100"
```

---

## Why?

Prevents one namespace from consuming all cluster resources.

Example:

```
Cluster

↓

Team-A

↓

20 CPU Maximum

----------------

Team-B

↓

20 CPU Maximum
```

---

## Verify

```bash
kubectl get resourcequota
```

Describe:

```bash
kubectl describe resourcequota production-quota
```

---

# 14. LimitRange

A **LimitRange** defines default and minimum/maximum resource values for containers in a namespace.

Example:

```yaml
apiVersion: v1
kind: LimitRange

metadata:
  name: default-limits

spec:
  limits:
  - type: Container

    default:
      cpu: "500m"
      memory: "512Mi"

    defaultRequest:
      cpu: "250m"
      memory: "256Mi"
```

---

## Benefit

If a developer omits resources:

```yaml
containers:
- name: app
  image: nginx
```

The defaults are automatically applied.

---

# 15. Namespace Resource Governance

Production clusters often combine:

```
Namespace

↓

LimitRange

↓

ResourceQuota

↓

Application Pods
```

Flow:

```
Developer

↓

Deploy Pod

↓

LimitRange Adds Defaults

↓

ResourceQuota Validates Usage

↓

Pod Admitted
```

This creates consistent and predictable resource management.

---

# 16. Resource Monitoring

Install the Metrics Server to collect resource usage.

Common commands:

```bash
kubectl top nodes
```

```bash
kubectl top pods
```

Example:

```
NAME         CPU   MEMORY

frontend     80m   120Mi

backend     450m   900Mi

database      2    4Gi
```

For long-term monitoring, tools such as **Prometheus** and **Grafana** are commonly used.

---

# 17. Production YAML

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

      containers:

      - name: payment

        image: payment:v2

        resources:

          requests:

            cpu: "500m"

            memory: "512Mi"

          limits:

            cpu: "1"

            memory: "1Gi"
```

---

# YAML Explained

## requests

Reserved resources used by the scheduler.

---

## limits

Maximum resources available to the container.

---

## Result

QoS:

```
Burstable
```

A common choice for production applications because it balances guaranteed capacity with the ability to burst when resources are available.

---

# 18. Part 2 Summary

You learned:

- CPU throttling
- OOMKilled
- QoS classes
- ResourceQuota
- LimitRange
- Namespace governance
- Resource monitoring
- Production Deployment YAML

You now understand how Kubernetes controls resource allocation and protects cluster stability.

# K8S-16 - Resource Management (Part 3)

> **Class Date:** 08-Jul-2026

---

# 📚 Table of Contents

19. Node Pressure & Pod Eviction
20. Requests vs Actual Usage
21. Vertical Pod Autoscaler (VPA)
22. Horizontal Pod Autoscaler (HPA) Overview
23. Common Mistakes
24. Troubleshooting
25. Production Best Practices
26. Hands-on Labs
27. CKA Exam Tips
28. Interview Questions
29. Resource Management Cheat Sheet
30. Chapter Summary

---

# 19. Node Pressure & Pod Eviction

When a node experiences resource pressure (especially memory, disk, or PID pressure), Kubernetes may evict Pods to protect node stability.

Common node conditions include:

- `MemoryPressure`
- `DiskPressure`
- `PIDPressure`

---

## Eviction Flow

```
Node

↓

MemoryPressure

↓

Eviction Manager

↓

Select Pods

↓

Evict Pods

↓

Free Resources
```

---

## Eviction Order

Eviction is influenced by:

1. QoS Class
2. Whether the Pod exceeds its resource requests
3. Pod priority (if PriorityClasses are used)

Typical QoS preference:

```
BestEffort

↓

Burstable

↓

Guaranteed
```

BestEffort Pods are generally the first candidates for eviction.

---

# 20. Requests vs Actual Usage

Requests reserve scheduling capacity.

Actual usage can differ significantly.

Example:

```
Request

500m CPU

↓

Actual Usage

150m CPU
```

The scheduler reserves based on **500m**, not the observed 150m.

---

## Why This Matters

Overestimated requests:

- Waste cluster capacity
- Increase cloud costs

Underestimated requests:

- Cause contention
- Increase eviction risk
- Lead to unstable workloads

---

## Production Sizing

Use monitoring data collected over time.

Review:

- Average usage
- Peak usage
- Seasonal spikes
- Deployment changes

---

# 21. Vertical Pod Autoscaler (VPA)

The **Vertical Pod Autoscaler** recommends or adjusts CPU and memory requests (and optionally limits, depending on configuration).

Architecture:

```
Metrics

↓

VPA Recommender

↓

Recommendation

↓

Updated Requests

↓

Pod Restart (when required)
```

---

## Example Use Cases

- Databases
- Internal services
- Long-running batch workers

---

## Important

VPA often requires Pods to be recreated when applying new resource values.

Because of this, carefully evaluate its use for workloads that cannot tolerate restarts.

---

# 22. Horizontal Pod Autoscaler (HPA) Overview

HPA scales **replica count**, not CPU or memory allocation.

Architecture:

```
CPU Utilization

↓

HPA

↓

Deployment

↓

Replicas

3

↓

6
```

---

## Example

```yaml
minReplicas: 2

maxReplicas: 10
```

If utilization increases:

```
Pods

2

↓

4

↓

6

↓

8
```

---

## HPA vs VPA

| Feature | HPA | VPA |
|---------|-----|-----|
| Scales Pods | ✅ | ❌ |
| Changes CPU/Memory Requests | ❌ | ✅ |
| Changes Replica Count | ✅ | ❌ |
| May Require Pod Restart | ❌ | Often Yes |

---

# 23. Common Mistakes

## No Requests

```yaml
resources: {}
```

Problem:

- Unpredictable scheduling
- BestEffort QoS
- First candidates for eviction

---

## No Limits

Without limits, a workload can consume more resources than intended, potentially affecting neighboring workloads.

Whether this is appropriate depends on your cluster policy and workload characteristics.

---

## Setting Limits Too Low

Example:

```
Memory Limit

256Mi

↓

Application Needs

600Mi

↓

OOMKilled
```

---

## Copying Resource Values

Avoid assigning identical requests and limits to every application.

Different workloads have different resource profiles.

Measure first.

---

# 24. Troubleshooting

## Check Resources

```bash
kubectl describe pod <pod-name>
```

Review:

```
Requests

Limits
```

---

## Detect OOMKilled

```bash
kubectl describe pod <pod-name>
```

Look for:

```
Reason: OOMKilled
```

---

## Monitor Usage

```bash
kubectl top pods
```

```bash
kubectl top nodes
```

---

## Check Events

```bash
kubectl get events \
  --sort-by=.metadata.creationTimestamp
```

---

## Check Node Conditions

```bash
kubectl describe node <node-name>
```

Review:

```
Conditions:

MemoryPressure
DiskPressure
PIDPressure
```

---

# Common Problems

| Problem | Possible Cause | Solution |
|----------|----------------|----------|
| Pod Pending | Requests exceed available node capacity | Resize requests or add cluster capacity |
| OOMKilled | Memory limit exceeded | Investigate usage, leaks, and sizing |
| Slow application | CPU throttling | Review CPU limits and actual utilization |
| Frequent evictions | Node resource pressure | Review QoS, requests, priorities, and node capacity |
| Low cluster utilization | Requests significantly overestimated | Right-size requests using monitoring data |

---

# 25. Production Best Practices

## Always Define Requests

Requests allow the scheduler to make informed placement decisions.

---

## Define Limits Carefully

Use limits where they fit your operational policy.

Avoid values that unnecessarily throttle or terminate healthy workloads.

---

## Monitor Before Tuning

Use:

- Metrics Server
- Prometheus
- Grafana

to understand real usage patterns.

---

## Review Regularly

Resource requirements evolve.

Revisit sizing after:

- New releases
- Traffic increases
- Architecture changes

---

## Use Autoscaling Wisely

- Use HPA for variable traffic.
- Consider VPA for workloads that benefit from automatic request sizing.
- Evaluate interactions carefully if combining autoscaling approaches.

---

# 26. Hands-on Labs

## Lab 1

Deploy a Pod with requests and limits.

Verify:

```bash
kubectl describe pod
```

---

## Lab 2

Observe usage:

```bash
kubectl top pods
```

Compare observed usage with configured requests.

---

## Lab 3

Create a ResourceQuota.

Verify that Pods exceeding the quota are rejected.

---

## Lab 4

Create a LimitRange.

Deploy a Pod without resource settings.

Verify default requests and limits are applied.

---

## Lab 5

Create an HPA.

Generate load and observe replica scaling.

---

# 27. CKA Exam Tips

✔ Memorize:

- Requests
- Limits
- QoS classes
- ResourceQuota
- LimitRange

✔ Practice:

```bash
kubectl top pods
kubectl top nodes
kubectl describe pod
kubectl describe node
```

✔ Understand:

- Scheduler uses requests.
- CPU is throttled.
- Memory limit violations lead to OOMKills.
- QoS affects eviction priority.

---

# 28. Interview Questions

## Beginner

1. What is a resource request?
2. What is a resource limit?
3. What happens when memory exceeds its limit?
4. What is CPU throttling?
5. Why are requests important?

---

## Intermediate

1. Explain QoS classes.
2. What is ResourceQuota?
3. What is LimitRange?
4. Why does a Pod remain Pending?
5. How does Kubernetes schedule Pods?

---

## Advanced

1. Explain Kubernetes eviction behavior.
2. Compare HPA and VPA.
3. Design a resource strategy for a multi-tenant cluster.
4. How would you troubleshoot frequent OOMKills?
5. How would you right-size application resources?

---

## Scenario-Based

### Scenario 1

A Pod is repeatedly OOMKilled.

What would you investigate before increasing the memory limit?

---

### Scenario 2

The cluster has low CPU utilization but many Pods remain Pending.

What configuration would you review first?

---

### Scenario 3

A namespace consumes most of the cluster resources.

Which Kubernetes resource can help control this?

---

### Scenario 4

A workload experiences variable traffic throughout the day.

Would HPA or VPA be more appropriate? Why?

---

### Scenario 5

A node reports `MemoryPressure`.

Which Pods are most likely to be evicted first?

---

# 29. Resource Management Cheat Sheet

Describe Pod:

```bash
kubectl describe pod <pod-name>
```

Show Pod usage:

```bash
kubectl top pods
```

Show node usage:

```bash
kubectl top nodes
```

Describe node:

```bash
kubectl describe node <node-name>
```

List quotas:

```bash
kubectl get resourcequota
```

List limit ranges:

```bash
kubectl get limitrange
```

Show events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# 30. Chapter Summary

Congratulations! 🎉

You have completed **K8S-16 – Resource Management**.

In this chapter, you learned:

- CPU requests
- Memory requests
- CPU limits
- Memory limits
- Scheduler behavior
- CPU throttling
- OOMKilled
- QoS classes
- ResourceQuota
- LimitRange
- Namespace governance
- Resource monitoring
- Node pressure
- Evictions
- HPA and VPA overview
- Troubleshooting
- Hands-on labs
- CKA preparation
- Interview questions

You now understand how Kubernetes allocates, protects, and optimizes compute resources for production workloads.

