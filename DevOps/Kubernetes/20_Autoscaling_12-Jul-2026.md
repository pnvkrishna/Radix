# K8S-20 - Autoscaling (HPA, VPA & Cluster Autoscaler) (Part 1)

> **Class Date:** 12-Jul-2026
>
> **Module:** Autoscaling


---

# 📚 Table of Contents

1. Introduction
2. Why Autoscaling?
3. Types of Autoscaling
4. Metrics Server
5. Horizontal Pod Autoscaler (HPA)
6. HPA Workflow
7. First HPA YAML
8. Part 1 Summary

---

# 1. Introduction

Application traffic changes over time.

Example:

```
09:00 AM

↓

100 Users

----------------

01:00 PM

↓

5,000 Users

----------------

11:00 PM

↓

50 Users
```

Running the same number of Pods all day is often inefficient.

Autoscaling allows Kubernetes to adjust resources based on demand.

---

# 2. Why Autoscaling?

Without autoscaling:

```
Traffic Spike

↓

CPU Usage 100%

↓

Slow Response

↓

User Complaints
```

With autoscaling:

```
Traffic Spike

↓

CPU Increases

↓

More Pods Created

↓

Load Distributed
```

Similarly, when demand decreases:

```
Traffic Drops

↓

Unused Pods Removed

↓

Lower Infrastructure Cost
```

---

# 3. Types of Autoscaling

Kubernetes supports multiple autoscaling mechanisms.

| Autoscaler | Scales | Typical Trigger |
|------------|--------|-----------------|
| Horizontal Pod Autoscaler (HPA) | Number of Pods | CPU, memory, custom or external metrics |
| Vertical Pod Autoscaler (VPA) | CPU & memory requests (and optionally limits) | Resource usage |
| Cluster Autoscaler | Number of Nodes | Pending Pods due to insufficient cluster capacity |

---

## Relationship

```
Application

↓

HPA

↓

Pods

↓

Cluster Autoscaler

↓

Nodes
```

HPA creates additional Pods.

If there is insufficient cluster capacity to run them, the Cluster Autoscaler may add more nodes.

---

# 4. Metrics Server

HPA requires metrics.

The Metrics Server collects short-term CPU and memory usage from the cluster.

Architecture:

```
Pods

↓

Kubelet

↓

Metrics Server

↓

HPA

↓

Scale Decision
```

---

## Verify

```bash
kubectl top nodes
```

```bash
kubectl top pods
```

If these commands return metrics, the Metrics Server is functioning.

---

# 5. Horizontal Pod Autoscaler (HPA)

HPA adjusts the **number of Pod replicas**.

Example:

```
Traffic

↓

CPU 85%

↓

HPA

↓

Replicas

3

↓

6
```

When demand decreases:

```
CPU 15%

↓

HPA

↓

Replicas

6

↓

3
```

---

## Common Metrics

- CPU utilization
- Memory utilization
- Custom metrics
- External metrics

---

# 6. HPA Workflow

```
Application

↓

CPU Usage

↓

Metrics Server

↓

HPA Controller

↓

Desired Replicas Calculated

↓

Deployment Updated

↓

Pods Created or Removed
```

---

## Example

Current:

```
Replicas

2
```

Target CPU:

```
70%
```

Observed CPU:

```
95%
```

Result:

```
Scale Up
```

---

# 7. First HPA YAML

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: payment-hpa

spec:

  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payment

  minReplicas: 2
  maxReplicas: 10

  metrics:

  - type: Resource

    resource:

      name: cpu

      target:

        type: Utilization

        averageUtilization: 70
```

---

# YAML Explained

## scaleTargetRef

Specifies the workload to scale.

---

## minReplicas

Minimum number of replicas.

---

## maxReplicas

Maximum number of replicas.

---

## averageUtilization

Target average CPU utilization across the Pods.

---

# Verify

Create:

```bash
kubectl apply -f hpa.yaml
```

View:

```bash
kubectl get hpa
```

Describe:

```bash
kubectl describe hpa payment-hpa
```

---

# Production Example

```
CPU

85%

↓

HPA

↓

Deployment

↓

Replicas

3 → 6
```

The Deployment controller then creates additional Pods.

---

# 8. Part 1 Summary

You learned:

- Why autoscaling is important
- Types of autoscaling
- Metrics Server
- Horizontal Pod Autoscaler
- HPA workflow
- Basic HPA YAML

You now understand how Kubernetes automatically scales the number of Pods based on resource utilization.

# K8S-20 - Autoscaling (HPA, VPA & Cluster Autoscaler) (Part 2)

> **Class Date:** 12-Jul-2026

---

# 📚 Table of Contents

9. HPA Scaling Algorithm
10. CPU vs Memory Scaling
11. Custom & External Metrics
12. HPA Scaling Behavior
13. Vertical Pod Autoscaler (VPA)
14. Cluster Autoscaler
15. Production Architecture
16. Part 2 Summary

---

# 9. HPA Scaling Algorithm

HPA continuously compares:

```
Current Metric

↓

Target Metric
```

If the observed value is higher than the target:

```
Scale Up
```

If it is lower:

```
Scale Down
```

---

## Simplified Formula

```
Desired Replicas

=

Current Replicas

×

(Current Metric ÷ Target Metric)
```

---

## Example 1

Current replicas:

```
4
```

Target CPU:

```
50%
```

Actual CPU:

```
100%
```

Calculation:

```
4 × (100 / 50)

=

8 Replicas
```

Result:

```
Scale

4

↓

8
```

---

## Example 2

Current replicas:

```
10
```

Target CPU:

```
50%
```

Actual CPU:

```
25%
```

Calculation:

```
10 × (25 / 50)

=

5 Replicas
```

Result:

```
Scale

10

↓

5
```

---

# Important Note

The HPA controller does **not** scale continuously every second.

It evaluates metrics periodically and respects:

- Stabilization windows
- Scaling policies
- Minimum replicas
- Maximum replicas

This prevents rapid oscillation ("thrashing").

---

# 10. CPU vs Memory Scaling

HPA can scale based on different resource metrics.

---

## CPU Scaling

Example:

```
Target CPU

70%
```

Observed:

```
92%
```

Result:

```
Scale Up
```

---

## Memory Scaling

Example:

```
Target Memory

75%
```

Observed:

```
82%
```

Result:

```
Scale Up
```

---

## Example YAML

```yaml
metrics:

- type: Resource

  resource:

    name: memory

    target:

      type: Utilization

      averageUtilization: 75
```

---

## CPU vs Memory

| CPU | Memory |
|------|---------|
| Responds quickly to load | Often changes more gradually |
| Good for web APIs | Useful for memory-intensive workloads |
| Common default | Use when memory drives performance |

---

# 11. Custom & External Metrics

Not every application should scale only on CPU.

Examples:

```
RabbitMQ Queue Length

Kafka Lag

HTTP Requests/Second

Active Sessions

Business Events
```

These are:

```
Custom Metrics
```

---

External systems can also provide metrics.

Examples:

- Cloud monitoring services
- Prometheus
- Managed message queues

---

## Architecture

```
Application

↓

Prometheus

↓

Custom Metrics Adapter

↓

HPA

↓

Scale
```

---

## Production Example

Queue size:

```
50

↓

No Scaling

----------------

500

↓

Scale Up
```

This is often more meaningful than CPU utilization for worker applications.

---

# 12. HPA Scaling Behavior

Modern HPA supports configurable behavior.

Example:

```yaml
behavior:

  scaleUp:

    stabilizationWindowSeconds: 0

  scaleDown:

    stabilizationWindowSeconds: 300
```

---

## Why Stabilization?

Traffic can fluctuate rapidly.

Without stabilization:

```
CPU

80

↓

20

↓

90

↓

30

↓

Pods

Up

↓

Down

↓

Up

↓

Down
```

This creates instability.

---

## With Stabilization

```
Traffic

↓

Temporary Drop

↓

Wait

↓

Still Low?

↓

Scale Down
```

This avoids unnecessary replica changes.

---

# 13. Vertical Pod Autoscaler (VPA)

Unlike HPA, VPA adjusts **resource requests**.

```
Before

CPU Request

250m

↓

After

500m
```

---

## Architecture

```
Pod

↓

Usage Analysis

↓

VPA

↓

Recommend

↓

Update Requests
```

---

## Modes

| Mode | Behavior |
|------|----------|
| Off | Recommendations only |
| Initial | Apply recommendations only when Pods are first created |
| Auto | Update resource requests (may recreate Pods depending on implementation) |

> **Note:** VPA may recreate Pods when applying updated resource requests. Consider workload disruption before enabling automatic updates.

---

## HPA + VPA

Running HPA and VPA together on the same CPU or memory metric can cause conflicting decisions.

Common production pattern:

```
HPA

↓

Scale Pods

----------------

VPA

↓

Recommend Resources
```

or use HPA for CPU and VPA recommendations for capacity planning.

---

# 14. Cluster Autoscaler

HPA scales Pods.

But what if:

```
No Node Has Enough Resources?
```

Pods remain:

```
Pending
```

Cluster Autoscaler solves this.

---

## Workflow

```
HPA

↓

Creates Pods

↓

Pods Pending

↓

Cluster Autoscaler

↓

Adds Node

↓

Pods Scheduled
```

---

## Scale Down

When nodes remain underutilized:

```
Unused Node

↓

Drain Workloads

↓

Remove Node
```

This reduces infrastructure costs.

---

# 15. Production Architecture

```
Users

↓

Ingress

↓

Deployment

↓

HPA

↓

Pods

↓

Cluster Autoscaler

↓

Nodes
```

Metrics flow:

```
Pods

↓

Kubelet

↓

Metrics Server

↓

HPA
```

Custom metrics flow:

```
Application

↓

Prometheus

↓

Metrics Adapter

↓

HPA
```

---

# Real-World Example

E-commerce Sale:

```
09:00

5 Pods

↓

12:00

25 Pods

↓

Extra Nodes Added

↓

18:00

Traffic Drops

↓

Pods Reduced

↓

Unused Nodes Removed
```

The platform automatically adapts without manual intervention.

---

# 16. Part 2 Summary

You learned:

- HPA scaling algorithm
- CPU-based scaling
- Memory-based scaling
- Custom metrics
- External metrics
- HPA stabilization
- VPA architecture
- Cluster Autoscaler
- Production autoscaling flow

You now understand how Kubernetes makes scaling decisions and how different autoscaling components work together.

# K8S-20 - Autoscaling (HPA, VPA & Cluster Autoscaler) (Part 3)

> **Class Date:** 12-Jul-2026

---

# 📚 Table of Contents

17. Troubleshooting HPA
18. KEDA (Kubernetes Event-Driven Autoscaling)
19. Common Autoscaling Mistakes
20. Production Best Practices
21. Hands-on Labs
22. CKA / CKAD Tips
23. Interview Questions
24. Autoscaling Cheat Sheet
25. Real Production Case Studies
26. Chapter Summary

---

# 17. Troubleshooting HPA

When autoscaling doesn't work as expected, troubleshoot systematically.

---

## Step 1: Check HPA

```bash
kubectl get hpa
```

Example output:

```text
NAME          REFERENCE               TARGETS    MINPODS   MAXPODS   REPLICAS
payment-hpa   Deployment/payment      82%/70%    2         10        5
```

---

## Step 2: Describe the HPA

```bash
kubectl describe hpa payment-hpa
```

Look for:

- Current metrics
- Desired replicas
- Scaling events
- Warning messages

---

## Step 3: Verify Metrics Server

```bash
kubectl top nodes
```

```bash
kubectl top pods
```

If these commands fail, verify the Metrics Server installation and health.

---

## Step 4: Check Deployment

```bash
kubectl get deployment payment
```

Verify:

- Desired replicas
- Available replicas
- Ready replicas

---

## Step 5: Check Events

```bash
kubectl get events --sort-by=.lastTimestamp
```

Look for:

- Failed scheduling
- Image pull failures
- Resource shortages

---

# Common Problems

| Problem | Possible Cause | Solution |
|----------|----------------|----------|
| HPA never scales | Metrics unavailable | Check Metrics Server |
| Pods stay Pending | No cluster capacity | Verify Cluster Autoscaler or add nodes |
| HPA scales but application remains slow | Application bottleneck | Profile the application, database, or downstream services |
| Constant scaling up/down | Aggressive scaling behavior | Configure stabilization windows and policies |
| No scaling despite high traffic | Wrong metric or target | Verify HPA metric configuration |

---

# 18. KEDA (Kubernetes Event-Driven Autoscaling)

KEDA extends Kubernetes autoscaling beyond CPU and memory.

It can scale workloads based on external events.

Examples:

- RabbitMQ queue length
- Kafka consumer lag
- Azure Service Bus
- AWS SQS
- Redis Streams
- Prometheus metrics

---

## Architecture

```
External Event

↓

KEDA

↓

HPA

↓

Deployment

↓

Pods
```

KEDA creates and manages an HPA behind the scenes for supported scalers.

---

## Example

Queue:

```
Messages = 0

↓

Replicas = 0
```

Later:

```
Messages = 5,000

↓

Replicas = 20
```

This allows event-driven workloads to scale efficiently.

---

## Common KEDA Use Cases

- Background workers
- Batch processing
- Message queue consumers
- Scheduled jobs
- Event-driven microservices

---

# 19. Common Autoscaling Mistakes

---

## Mistake 1

Using HPA without resource requests.

❌

```yaml
resources: {}
```

Without CPU or memory requests, utilization-based HPA cannot make accurate decisions.

---

## Mistake 2

Setting:

```
minReplicas = maxReplicas
```

This effectively disables scaling.

---

## Mistake 3

Using only CPU metrics.

Some applications are constrained by:

- Memory
- Queue length
- Database throughput
- External events

Choose metrics that reflect the application's bottleneck.

---

## Mistake 4

Aggressive scale-down.

Immediate scale-down can create instability.

Use stabilization windows to reduce oscillation.

---

## Mistake 5

Ignoring startup time.

If a Pod requires:

```
3 minutes

↓

Ready
```

Scaling reacts more slowly than expected.

Consider:

- Startup probes
- Readiness probes
- Realistic autoscaling thresholds

---

# 20. Production Best Practices

---

## Define Resource Requests

Example:

```yaml
resources:

  requests:

    cpu: 250m

    memory: 256Mi
```

These are essential for accurate scheduling and CPU utilization calculations.

---

## Use Reasonable Limits

Avoid extremely high limits that allow excessive resource consumption or overly restrictive limits that cause throttling.

---

## Set Realistic HPA Targets

Typical starting points:

- CPU: 60–80%
- Memory: workload dependent

Monitor real production behavior before tuning.

---

## Protect Against Thrashing

Configure:

- Stabilization windows
- Scaling policies

to avoid frequent scaling events.

---

## Monitor Autoscaling

Track:

- Replica count
- CPU utilization
- Memory utilization
- Scaling events
- Response time
- Error rate

Autoscaling should improve user experience, not just resource utilization.

---

# 21. Hands-on Labs

---

## Lab 1

Deploy:

```yaml
Deployment
```

Create:

```yaml
HPA
```

Generate load using:

```bash
kubectl run load-generator \
  --image=busybox:1.36 \
  -it --rm -- sh
```

Install `wget` or use another HTTP client if needed, then send repeated requests to the service.

Observe:

```bash
kubectl get hpa -w
```

---

## Lab 2

Watch Deployment replicas:

```bash
kubectl get deployment -w
```

Observe scaling behavior.

---

## Lab 3

Generate traffic.

Watch:

```bash
kubectl top pods
```

Observe CPU utilization.

---

## Lab 4

Reduce traffic.

Observe gradual scale-down.

---

## Lab 5

If using a cloud provider that supports Cluster Autoscaler:

- Create resource pressure
- Observe Pending Pods
- Watch new nodes join the cluster

---

# 22. CKA / CKAD Tips

✔ Know the differences:

- HPA
- VPA
- Cluster Autoscaler
- KEDA

✔ Practice:

```bash
kubectl get hpa
kubectl describe hpa
kubectl top nodes
kubectl top pods
```

✔ Remember:

- HPA scales Pods.
- VPA adjusts resource requests.
- Cluster Autoscaler scales Nodes.
- KEDA enables event-driven autoscaling.

---

# 23. Interview Questions

## Beginner

1. What is HPA?
2. What metrics can HPA use?
3. What is Metrics Server?
4. What is VPA?
5. What is Cluster Autoscaler?

---

## Intermediate

1. Explain how HPA calculates desired replicas.
2. Why are CPU requests important for HPA?
3. Compare HPA and VPA.
4. When would you use custom metrics?
5. What problem does KEDA solve?

---

## Advanced

1. Design autoscaling for an e-commerce application.
2. Design autoscaling for Kafka consumers.
3. How would you prevent autoscaling thrashing?
4. Explain how HPA and Cluster Autoscaler work together.
5. Troubleshoot an HPA that never scales despite increased load.

---

## Scenario-Based

### Scenario 1

A Deployment scales to its maximum replicas, but some Pods remain Pending.

What should you investigate?

---

### Scenario 2

CPU usage is low, but RabbitMQ queues continue growing.

Would CPU-based HPA be sufficient?

---

### Scenario 3

An application experiences brief traffic spikes.

How would you avoid unnecessary scale-up and scale-down cycles?

---

### Scenario 4

A workload processes messages only during business hours.

Would KEDA be an appropriate solution?

---

### Scenario 5

A memory-intensive application frequently encounters OOMKills.

Would HPA alone solve the problem?

---

# 24. Autoscaling Cheat Sheet

View HPA:

```bash
kubectl get hpa
```

Describe HPA:

```bash
kubectl describe hpa
```

Show Pod metrics:

```bash
kubectl top pods
```

Show Node metrics:

```bash
kubectl top nodes
```

Watch replicas:

```bash
kubectl get deployment -w
```

Watch HPA:

```bash
kubectl get hpa -w
```

---

# 25. Real Production Case Studies

## E-Commerce Platform

```
Flash Sale

↓

Traffic ×10

↓

HPA

↓

Pods

↓

Cluster Autoscaler

↓

Additional Nodes
```

Result:

- Maintained response times
- Reduced manual intervention

---

## Video Processing

```
Upload Queue

↓

KEDA

↓

Worker Pods

↓

Process Videos

↓

Queue Empty

↓

Scale to Zero
```

Result:

- Lower infrastructure costs
- Efficient handling of bursty workloads

---

## Banking API

```
Business Hours

↓

High CPU

↓

HPA

↓

More API Pods

↓

Evening

↓

Reduced Traffic

↓

Scale Down
```

Result:

- Better resource utilization while maintaining availability

---

# 26. Chapter Summary

Congratulations! 🎉

You have completed **K8S-20 – Autoscaling**.

In this chapter, you learned:

- Horizontal Pod Autoscaler (HPA)
- Metrics Server
- HPA scaling algorithm
- CPU and memory scaling
- Custom metrics
- External metrics
- Vertical Pod Autoscaler (VPA)
- Cluster Autoscaler
- KEDA
- Troubleshooting
- Production best practices
- Hands-on labs
- CKA / CKAD preparation
- Interview questions

You now understand how Kubernetes automatically scales applications and cluster infrastructure to meet changing demand while balancing performance, availability, and cost.

