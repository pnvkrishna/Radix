# Kubernetes Class 06 - ReplicaSets & Deployments (Part 1)

> **Class Date:** 28-Jun-2026
>
> **Module:** ReplicaSets & Deployments
>
> **File Name:** `K8S-06_ReplicaSets-and-Deployments_28-Jun-2026.md`

---

# 📚 Table of Contents

1. Introduction
2. Why Pods Are Not Enough
3. What is a ReplicaSet?
4. ReplicaSet Architecture
5. Self-Healing
6. Scaling Applications
7. ReplicaSet YAML Explained
8. Labels & Selectors
9. Deployment Introduction
10. Deployment Architecture
11. Summary

---

# 1. Introduction

In the previous chapter, we learned that a Pod is the smallest deployable unit in Kubernetes.

However, Pods have one major limitation.

If a Pod crashes, is deleted accidentally, or the node hosting it fails, the Pod may disappear permanently unless another Kubernetes resource manages it.

This is where **ReplicaSets** and **Deployments** become essential.

They provide:

- High Availability
- Self-Healing
- Automatic Scaling
- Rolling Updates
- Rollbacks

Almost every production application in Kubernetes is deployed using a **Deployment**, which internally manages a **ReplicaSet**, which in turn manages one or more **Pods**.

---

# 2. Why Pods Are Not Enough

Imagine you create a Pod:

```bash
kubectl run nginx --image=nginx
```

Current state:

```
Node

↓

Pod

↓

Nginx
```

Now suppose someone deletes the Pod:

```bash
kubectl delete pod nginx
```

Result:

```
Pod Deleted

↓

Application Offline
```

Kubernetes will **not** recreate this Pod automatically because no controller is managing it.

This creates downtime.

---

## Production Problem

Consider an e-commerce application.

```
Customer

↓

Website

↓

Pod Deleted

↓

Website Down

↓

Business Loss
```

Production systems cannot rely on manually recreated Pods.

They require automatic recovery.

---

# 3. What is a ReplicaSet?

A ReplicaSet is a Kubernetes controller responsible for ensuring that a specified number of identical Pod replicas are always running.

Instead of managing Pods directly, you tell Kubernetes:

> "Always keep 3 Pods running."

The ReplicaSet continuously monitors the cluster and maintains the desired number of Pods.

---

## Example

Desired replicas:

```
3 Pods
```

Current state:

```
Pod-1 ✅

Pod-2 ✅

Pod-3 ❌ Deleted
```

ReplicaSet detects the difference:

```
Desired = 3

Running = 2

↓

Creates New Pod

↓

Running = 3
```

This automatic recovery is called **Self-Healing**.

---

# 4. ReplicaSet Architecture

```
                 Deployment
                      │
                      ▼
               ReplicaSet Controller
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
        Pod-1       Pod-2       Pod-3
          │           │           │
      Container   Container   Container
```

Notice that in production, users typically create a **Deployment**, not a ReplicaSet directly.

The Deployment creates and manages the ReplicaSet.

---

# 5. Self-Healing

Self-healing is one of Kubernetes' most powerful features.

### Scenario

ReplicaSet configuration:

```
Replicas = 3
```

Current state:

```
Pod-1

Pod-2

Pod-3
```

Delete one Pod:

```bash
kubectl delete pod pod-2
```

What happens?

```
ReplicaSet Notices

↓

Current = 2

Desired = 3

↓

Creates New Pod

↓

Application Continues Running
```

Users experience little or no downtime.

---

# 6. Scaling Applications

ReplicaSets also make scaling simple.

Suppose traffic increases during a sale.

Current state:

```
Replicas = 3
```

Increase replicas:

```
Replicas = 6
```

Result:

```
Pod-1

Pod-2

Pod-3

Pod-4

Pod-5

Pod-6
```

Traffic is now distributed across more Pods.

When demand decreases:

```
Replicas = 2
```

ReplicaSet safely removes the extra Pods.

---

# Horizontal Scaling

ReplicaSets perform **Horizontal Scaling**.

```
Before

Pod

Pod

Pod

↓

After

Pod

Pod

Pod

Pod

Pod

Pod
```

Instead of making one server larger, Kubernetes adds more Pod replicas.

---

# 7. ReplicaSet YAML

Example:

```yaml
apiVersion: apps/v1

kind: ReplicaSet

metadata:
  name: nginx-rs

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:

    metadata:
      labels:
        app: nginx

    spec:

      containers:

      - name: nginx

        image: nginx
```

---

## Line-by-Line Explanation

### apiVersion

```yaml
apps/v1
```

ReplicaSets belong to the `apps/v1` API group.

---

### kind

```yaml
ReplicaSet
```

Specifies the Kubernetes resource type.

---

### metadata

Stores information such as:

- Name
- Labels
- Annotations

---

### replicas

```yaml
replicas: 3
```

The desired number of running Pods.

---

### selector

The ReplicaSet uses a selector to determine which Pods it manages.

```yaml
selector:
  matchLabels:
    app: nginx
```

---

### template

Defines the Pod template.

Every new Pod created by the ReplicaSet uses this template.

---

### containers

Defines the application container.

```yaml
containers:
- name: nginx
  image: nginx
```

---

# Create ReplicaSet

```bash
kubectl apply -f replicaset.yaml
```

---

# Verify ReplicaSet

```bash
kubectl get rs
```

Example output:

```
NAME

nginx-rs

DESIRED

3

CURRENT

3

READY

3
```

---

# Verify Pods

```bash
kubectl get pods
```

You should see:

```
nginx-rs-abc12

nginx-rs-def34

nginx-rs-ghi56
```

Each Pod has a unique generated name.

---

# 8. Labels & Selectors

ReplicaSets rely on labels.

Example:

```yaml
labels:
  app: nginx
```

Selector:

```yaml
matchLabels:
  app: nginx
```

Relationship:

```
ReplicaSet

↓

Selector

↓

app=nginx

↓

Matching Pods
```

If labels do not match, the ReplicaSet will not manage the Pods.

This is one of the most common configuration mistakes.

---

# 9. Introduction to Deployments

Although ReplicaSets are powerful, Kubernetes administrators rarely create them directly.

Instead, they use **Deployments**.

A Deployment is a higher-level controller that manages ReplicaSets.

It adds features such as:

- Rolling Updates
- Rollbacks
- Revision History
- Easy Scaling
- Declarative Updates

---

# Deployment Hierarchy

```
Deployment

↓

ReplicaSet

↓

Pods

↓

Containers
```

Think of it like this:

- Deployment = Manager
- ReplicaSet = Supervisor
- Pod = Worker
- Container = Employee

---

# 10. Why Deployments?

Suppose your application currently runs:

```
nginx:1.25
```

A new version is released:

```
nginx:1.26
```

Deployment updates the Pods gradually.

```
Old Pod

↓

New Pod

↓

Old Pod Removed

↓

Repeat
```

This process is called a **Rolling Update**.

Users continue accessing the application while the update is happening.

---

# 11. Part 1 Summary

In this chapter, you learned:

- Why Pods alone are insufficient
- What a ReplicaSet is
- How self-healing works
- Horizontal scaling
- ReplicaSet architecture
- ReplicaSet YAML explained line by line
- Labels and selectors
- Why Deployments are preferred over ReplicaSets
- Deployment architecture

The next section will cover Deployments in depth, including rolling updates, rollbacks, deployment strategies, YAML, scaling, troubleshooting, and production best practices.

---

# 12. What is a Deployment?

A **Deployment** is a Kubernetes controller that manages the complete lifecycle of an application.

Instead of creating Pods directly, you create a Deployment.

The Deployment automatically creates and manages:

- ReplicaSets
- Pods

This provides:

- High Availability
- Self-Healing
- Scaling
- Rolling Updates
- Rollbacks
- Version History

---

# Deployment Architecture

```
                    Deployment
                         │
                         ▼
                  ReplicaSet
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       Pod-1          Pod-2          Pod-3
          │              │              │
       Container      Container      Container
```

The Deployment is the top-level controller.

---

# Why Use Deployments?

Imagine an online shopping application.

Requirements:

- Always available
- Handle heavy traffic
- Upgrade without downtime
- Roll back quickly if something fails

Deployments provide all these capabilities.

---

# 13. Deployment YAML

Example:

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: nginx-deployment

spec:

  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:

    metadata:
      labels:
        app: nginx

    spec:

      containers:

      - name: nginx

        image: nginx:latest

        ports:
        - containerPort: 80
```

---

# YAML Explanation

## apiVersion

```yaml
apps/v1
```

Defines the API group.

---

## kind

```yaml
Deployment
```

Specifies the Kubernetes resource type.

---

## metadata

Stores:

- Name
- Labels
- Annotations

---

## replicas

```yaml
replicas: 3
```

Three Pods should always remain available.

---

## selector

Matches Pods using labels.

```yaml
matchLabels:
  app: nginx
```

---

## template

Defines the Pod template.

Every Pod created by the Deployment uses this template.

---

## containers

Application container configuration.

```yaml
containers:

- name: nginx

  image: nginx:latest
```

---

# Create Deployment

```bash
kubectl apply -f deployment.yaml
```

---

# Verify Deployment

```bash
kubectl get deployments
```

Example:

```
NAME

nginx-deployment

READY

3/3
```

---

# Verify ReplicaSets

```bash
kubectl get rs
```

---

# Verify Pods

```bash
kubectl get pods
```

---

# Complete Relationship

```
Deployment

↓

ReplicaSet

↓

Pods

↓

Containers
```

---

# 14. Scaling Deployments

Suppose traffic suddenly increases.

Current

```
Replicas = 3
```

Scale to

```
Replicas = 6
```

Command

```bash
kubectl scale deployment nginx-deployment \
--replicas=6
```

Result

```
Deployment

↓

ReplicaSet

↓

6 Pods Running
```

---

## Scale Down

```bash
kubectl scale deployment nginx-deployment \
--replicas=2
```

ReplicaSet safely removes extra Pods.

---

# Verify Scaling

```bash
kubectl get deployment
```

Expected

```
READY

6/6
```

---

# 15. Rolling Updates

One of Kubernetes' best features.

Suppose

Current

```
nginx:1.24
```

New Version

```
nginx:1.25
```

Update

```bash
kubectl set image deployment/nginx-deployment \
nginx=nginx:1.25
```

---

## Rolling Update Workflow

```
Old Pod

↓

New Pod Created

↓

Health Check

↓

Old Pod Removed

↓

Repeat
```

This continues until all Pods run the new version.

Users experience little or no downtime.

---

# Verify Rolling Update

```bash
kubectl rollout status deployment nginx-deployment
```

Output

```
deployment successfully rolled out
```

---

# View Rollout History

```bash
kubectl rollout history deployment nginx-deployment
```

Example

```
Revision 1

Revision 2

Revision 3
```

---

# 16. Rollbacks

Imagine the new application version contains a bug.

Deployment makes recovery easy.

Rollback

```bash
kubectl rollout undo deployment nginx-deployment
```

Workflow

```
Version 2

↓

Problem Detected

↓

Rollback

↓

Version 1 Restored
```

Application becomes stable again.

---

# Rollback to Specific Revision

View history

```bash
kubectl rollout history deployment nginx-deployment
```

Rollback

```bash
kubectl rollout undo deployment nginx-deployment \
--to-revision=2
```

---

# 17. Rolling Update Strategy

Default Strategy

```
RollingUpdate
```

Deployment gradually replaces old Pods.

```
Old Old Old

↓

New Old Old

↓

New New Old

↓

New New New
```

No downtime.

---

## Recreate Strategy

Deletes all old Pods first.

Then creates new Pods.

Workflow

```
Old Pods

↓

Deleted

↓

New Pods Created
```

Simple

But

Application downtime occurs.

---

## YAML Example

```yaml
strategy:

  type: RollingUpdate
```

or

```yaml
strategy:

  type: Recreate
```

---

# RollingUpdate Parameters

```yaml
rollingUpdate:

  maxUnavailable: 1

  maxSurge: 1
```

---

## maxUnavailable

Maximum Pods that may be unavailable.

Example

```
3 Pods

↓

1 May Stop

↓

2 Continue Running
```

---

## maxSurge

Extra Pods created during upgrade.

Example

```
Desired = 3

Surge = 1

↓

4 Pods Temporarily
```

Faster deployment.

---

# 18. Update Deployment Using YAML

Modify

```yaml
image:

nginx:1.26
```

Apply

```bash
kubectl apply -f deployment.yaml
```

Deployment automatically performs a Rolling Update.

---

# Watch Deployment

```bash
kubectl get pods -w
```

Observe

Old Pods disappear.

New Pods appear.

---

# Pause Rollout

```bash
kubectl rollout pause deployment nginx-deployment
```

Useful during maintenance.

---

# Resume Rollout

```bash
kubectl rollout resume deployment nginx-deployment
```

Deployment continues.

---

# Restart Deployment

Restart Pods without changing YAML.

```bash
kubectl rollout restart deployment nginx-deployment
```

Useful after updating ConfigMaps or Secrets.

---

# 19. Common Commands

List Deployments

```bash
kubectl get deployments
```

Describe Deployment

```bash
kubectl describe deployment nginx-deployment
```

Delete Deployment

```bash
kubectl delete deployment nginx-deployment
```

View Pods

```bash
kubectl get pods
```

View ReplicaSets

```bash
kubectl get rs
```

---

# Part 2 Summary

In this section, you learned:

- What a Deployment is
- Deployment architecture
- Deployment YAML explained line by line
- Scaling applications
- Rolling Updates
- Rollbacks
- Deployment strategies
- RollingUpdate parameters
- Rollout commands
- Deployment management commands

Deployments are the standard way to deploy stateless applications in Kubernetes. They provide reliability, automation, and safe application updates, making them one of the most important resources for every Kubernetes administrator.

---

# 20. Deployment Strategies

Deploying a new application version is more than simply replacing containers. Different strategies balance availability, risk, and deployment speed.

The most common deployment strategies are:

- Recreate
- Rolling Update
- Blue-Green
- Canary

---

# Recreate Strategy

The old version is completely removed before the new version starts.

```
Version 1

██████████

↓

Delete

↓

Version 2

██████████
```

### Advantages

- Simple
- Easy to understand
- No mixed application versions

### Disadvantages

- Application downtime
- Not suitable for production web applications

### Best For

- Internal tools
- Development environments
- Scheduled maintenance windows

---

# Rolling Update Strategy

(Default Kubernetes Strategy)

Pods are updated gradually.

```
V1 V1 V1 V1

↓

V2 V1 V1 V1

↓

V2 V2 V1 V1

↓

V2 V2 V2 V1

↓

V2 V2 V2 V2
```

### Advantages

- No downtime
- Safe updates
- Default Kubernetes behavior

### Disadvantages

- Two application versions may run simultaneously during the update
- Database compatibility should be considered

---

# Blue-Green Deployment

Maintain two identical environments.

```
Blue Environment

↓

Current Production

--------------------

Green Environment

↓

New Version
```

Switch traffic only after testing.

```
Users

↓

Blue

↓

Switch

↓

Green
```

### Advantages

- Fast rollback
- Very low downtime
- Easy testing before release

### Disadvantages

- Requires twice the infrastructure during deployment

### Common Use Cases

- Banking
- E-commerce
- Enterprise applications

---

# Canary Deployment

Release the new version to a small percentage of users first.

```
100 Users

↓

95 → Version 1

5 → Version 2
```

If successful:

```
50%

↓

75%

↓

100%
```

### Advantages

- Lower deployment risk
- Real-user validation
- Early issue detection

### Disadvantages

- More complex traffic management
- Often implemented with Ingress Controllers or Service Meshes (such as Istio or Linkerd)

---

# Strategy Comparison

| Strategy | Downtime | Rollback | Complexity | Production |
|-----------|-----------|----------|------------|------------|
| Recreate | High | Easy | Low | Rare |
| Rolling Update | None | Easy | Medium | Very Common |
| Blue-Green | Very Low | Very Easy | Medium | Common |
| Canary | None | Excellent | High | Large Enterprises |

---

# 21. Self-Healing in Deployments

Deployments continuously maintain the desired application state.

Suppose:

```
Desired Replicas

=

3
```

Running Pods:

```
Pod-1

Pod-2

Pod-3
```

Now Node-2 crashes.

```
Node Failure

↓

Pod Lost
```

ReplicaSet detects:

```
Desired = 3

Current = 2
```

Scheduler automatically creates another Pod on a healthy node.

```
Healthy Node

↓

New Pod

↓

Application Restored
```

This automatic recovery is called **Self-Healing**.

---

# 22. Deployment Lifecycle

```
Developer

↓

Git Commit

↓

CI/CD Pipeline

↓

Docker Image

↓

Container Registry

↓

Deployment YAML Updated

↓

kubectl apply

↓

Deployment

↓

ReplicaSet

↓

Pods

↓

Running Application
```

This workflow is commonly used with Jenkins, GitHub Actions, GitLab CI/CD, or Azure DevOps.

---

# 23. Deployment Troubleshooting

## Check Deployments

```bash
kubectl get deployments
```

---

## Describe Deployment

```bash
kubectl describe deployment nginx-deployment
```

Useful for viewing:

- Events
- Replica counts
- Conditions
- Rollout progress

---

## Check ReplicaSets

```bash
kubectl get rs
```

---

## Check Pods

```bash
kubectl get pods
```

---

## View Pod Details

```bash
kubectl describe pod POD_NAME
```

---

## View Application Logs

```bash
kubectl logs POD_NAME
```

---

## Check Rollout Status

```bash
kubectl rollout status deployment nginx-deployment
```

---

# Common Deployment Issues

## Pods Not Starting

Possible causes:

- Invalid container image
- Missing ConfigMap or Secret
- Application startup failure
- Insufficient resources

---

## ImagePullBackOff

Possible causes:

- Incorrect image name
- Private registry authentication issue
- Image not found

---

## Deployment Stuck

Possible causes:

- Failed readiness probe
- Insufficient cluster resources
- Scheduling constraints

---

## CrashLoopBackOff

Possible causes:

- Application crashes immediately
- Configuration errors
- Missing dependencies

---

# 24. Hands-on Labs

## Lab 1

Create a Deployment:

```bash
kubectl create deployment nginx --image=nginx
```

---

## Lab 2

Scale to five replicas:

```bash
kubectl scale deployment nginx --replicas=5
```

Verify:

```bash
kubectl get pods
```

---

## Lab 3

Update the image:

```bash
kubectl set image deployment/nginx nginx=nginx:1.27
```

Watch the rollout:

```bash
kubectl rollout status deployment nginx
```

---

## Lab 4

Rollback:

```bash
kubectl rollout undo deployment nginx
```

---

## Lab 5

Delete one Pod manually:

```bash
kubectl delete pod POD_NAME
```

Observe that the Deployment recreates it automatically.

---

# 25. Production Best Practices

✅ Always use Deployments instead of standalone Pods.

✅ Store Deployment YAML files in Git.

✅ Use meaningful labels and annotations.

✅ Avoid using the `latest` image tag in production. Prefer immutable version tags (for example, `nginx:1.27.2`) for predictable deployments and easier rollbacks.

✅ Configure readiness and liveness probes.

✅ Define CPU and memory requests and limits.

✅ Use Rolling Updates for stateless applications.

✅ Test rollbacks before production releases.

✅ Monitor deployment health using Prometheus and Grafana.

---

# 26. CKA Exam Tips

✔ Create Deployments quickly.

✔ Scale Deployments using both YAML and `kubectl scale`.

✔ Practice image updates.

✔ Know rollout commands.

✔ Understand rollback procedures.

✔ Read Deployment YAML comfortably.

---

# 27. Real Production Scenario

An e-commerce application is running with:

```
Replicas = 10
```

A new version is released.

Deployment performs a Rolling Update.

During the update:

```
Version 1

↓

Version 2
```

One new Pod repeatedly fails its readiness checks.

Kubernetes:

- Keeps the failed Pod out of Service traffic
- Continues serving users with healthy Pods
- Reports the rollout issue

The operations team investigates, fixes the problem, and redeploys—or rolls back if necessary.

This minimizes customer impact.

---

# 28. Interview Questions

## Basic

1. What is a Deployment?
2. Why use Deployments instead of Pods?
3. What is a ReplicaSet?
4. What is self-healing?
5. How do you scale a Deployment?

---

## Intermediate

1. Explain the Deployment architecture.
2. What is a Rolling Update?
3. How does a rollback work?
4. Difference between ReplicaSet and Deployment.
5. Explain `maxSurge` and `maxUnavailable`.

---

## Advanced

1. Explain Blue-Green Deployment.
2. Explain Canary Deployment.
3. How does Kubernetes achieve zero-downtime deployments?
4. How do readiness probes affect rolling updates?
5. Describe a production deployment workflow.

---

## Scenario-Based

### Scenario 1

A rollout is stuck because new Pods never become Ready.

How would you investigate?

---

### Scenario 2

After deploying a new version, users report failures.

How would you decide between fixing forward and rolling back?

---

### Scenario 3

A ReplicaSet shows fewer Ready Pods than desired.

What checks would you perform?

---

### Scenario 4

A Deployment update finished successfully, but some users still experience errors.

What Kubernetes and application-level factors would you examine?

---

### Scenario 5

Your cluster has limited capacity.

How would `maxSurge` and `maxUnavailable` affect your rollout strategy?

---

# 29. Deployment Cheat Sheet

Create

```bash
kubectl create deployment nginx --image=nginx
```

Scale

```bash
kubectl scale deployment nginx --replicas=5
```

Update Image

```bash
kubectl set image deployment/nginx nginx=nginx:1.27
```

Status

```bash
kubectl rollout status deployment nginx
```

History

```bash
kubectl rollout history deployment nginx
```

Rollback

```bash
kubectl rollout undo deployment nginx
```

Restart

```bash
kubectl rollout restart deployment nginx
```

Describe

```bash
kubectl describe deployment nginx
```

Delete

```bash
kubectl delete deployment nginx
```

---

# 30. Key Takeaways

- Deployments are the recommended way to manage stateless applications.
- Deployments create and manage ReplicaSets, which in turn manage Pods.
- ReplicaSets provide self-healing and maintain the desired number of replicas.
- Rolling Updates enable near zero-downtime application upgrades.
- Rollbacks allow rapid recovery from failed releases.
- Blue-Green and Canary strategies further reduce deployment risk in production.
- Proper probes, resource requests, and monitoring improve deployment reliability.

---

# 31. Chapter Summary

Congratulations! 🎉

You have mastered one of the most important Kubernetes concepts.

In this chapter, you learned:

- Why Pods are not enough
- ReplicaSet architecture
- Self-healing
- Horizontal scaling
- Deployment architecture
- Deployment YAML
- Rolling Updates
- Rollbacks
- Deployment strategies
- Production best practices
- Troubleshooting
- Hands-on labs
- CKA preparation
- Interview questions

These concepts form the foundation of modern Kubernetes application delivery.

In the next chapter, you'll learn **Services**—how applications communicate reliably inside and outside the cluster, even as Pods are created, deleted, and replaced.