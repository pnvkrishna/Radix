# K8S-15 - Labels, Selectors & Annotations (Part 1)

> **Class Date:** 07-Jul-2026
>
> **Module:** Labels, Selectors & Annotations
>
> **File Name:** `K8S-15_Labels_Selectors_Annotations_07-Jul-2026.md`

---

# 📚 Table of Contents

1. Introduction
2. Why Labels?
3. What are Labels?
4. Label Rules
5. Recommended Kubernetes Labels
6. What are Selectors?
7. Equality-based Selectors
8. First Label YAML
9. Part 1 Summary

---

# 1. Introduction

Imagine a Kubernetes cluster running hundreds or thousands of Pods.

```
Cluster

├── frontend-1
├── frontend-2
├── backend-1
├── backend-2
├── payment-1
├── payment-2
├── mysql-0
├── redis-0
├── prometheus
├── grafana
├── fluent-bit
└── node-exporter
```

Without labels, Kubernetes would have no structured way to identify related resources.

Questions like these become difficult:

- Which Pods belong to the frontend?
- Which Pods should receive traffic?
- Which Pods should a Deployment manage?
- Which Pods should Prometheus monitor?

Labels solve this problem.

---

# 2. Why Labels?

Labels are lightweight **key-value pairs** attached to Kubernetes resources.

Think of them like tags on files.

Example:

```
Employee

↓

Department = HR

Location = Hyderabad

Experience = Senior
```

Similarly:

```
Pod

↓

app=frontend

environment=production

version=v1
```

Now Kubernetes can easily identify and group resources.

---

# 3. What are Labels?

A label consists of:

```
Key

↓

Value
```

Example:

```
app=frontend

environment=production

tier=web

version=v1
```

A resource can have many labels.

Example:

```yaml
metadata:
  labels:
    app: frontend
    environment: production
    tier: web
    version: v1
```

---

# Why Multiple Labels?

Different systems may use different labels.

Example:

```
Service

↓

app=frontend

----------------

Network Policy

↓

tier=web

----------------

Monitoring

↓

environment=production
```

One Pod can satisfy all of these independently.

---

# 4. Label Rules

Label keys should be meaningful and stable.

Examples:

```
app

version

environment

component

team

region
```

Avoid labels that change frequently, such as timestamps or random values, because they can make selection and automation difficult.

---

## Good Labels

```yaml
app: payment
environment: production
team: platform
version: v2
```

---

## Poor Labels

```yaml
time: "2026-07-07T12:34:56Z"
random: "abc123"
```

These are not useful for selecting workloads.

---

# 5. Recommended Kubernetes Labels

The Kubernetes project recommends a common labeling convention.

| Label | Purpose |
|--------|----------|
| `app.kubernetes.io/name` | Application name |
| `app.kubernetes.io/instance` | Instance identifier |
| `app.kubernetes.io/version` | Application version |
| `app.kubernetes.io/component` | Component (frontend, backend, database) |
| `app.kubernetes.io/part-of` | Larger application or platform |
| `app.kubernetes.io/managed-by` | Tool managing the resource (Helm, Argo CD, etc.) |

---

## Example

```yaml
metadata:
  labels:
    app.kubernetes.io/name: payment
    app.kubernetes.io/version: "2.0"
    app.kubernetes.io/component: backend
    app.kubernetes.io/part-of: ecommerce
    app.kubernetes.io/managed-by: helm
```

Using these labels improves consistency across tools and teams.

---

# 6. What are Selectors?

Labels identify resources.

Selectors **find** resources.

```
Pod

↓

app=frontend

↓

Selector

↓

app=frontend

↓

Match Found
```

Selectors are used by many Kubernetes resources, including:

- Services
- Deployments
- ReplicaSets
- Jobs
- Network Policies

---

# 7. Equality-based Selectors

The simplest selector matches exact label values.

---

## Equal (`=` or `==`)

Example:

```
app=frontend
```

Matches:

```
frontend-1

frontend-2

frontend-3
```

---

## Not Equal (`!=`)

Example:

```
environment!=production
```

Matches every resource **except** those labeled `environment=production`.

---

## Examples

List Pods:

```bash
kubectl get pods -l app=frontend
```

Multiple labels:

```bash
kubectl get pods -l app=frontend,environment=production
```

Exclude production:

```bash
kubectl get pods -l environment!=production
```

---

# 8. First Label YAML

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: frontend

  labels:
    app: frontend
    environment: production
    version: v1

spec:
  containers:
  - name: nginx
    image: nginx:1.27
```

---

## Verify Labels

```bash
kubectl get pods --show-labels
```

---

Filter by label:

```bash
kubectl get pods -l app=frontend
```

---

Describe a Pod:

```bash
kubectl describe pod frontend
```

Look for:

```
Labels:
```

---

# Architecture

```
Pod

↓

Labels

├── app=frontend
├── version=v1
├── environment=production

↓

Selector

↓

Matching Resources
```

---

# Production Example

```
Service

↓

Selector

↓

app=payment

↓

payment-1

payment-2

payment-3
```

The Service sends traffic only to Pods matching its selector.

---

# 9. Part 1 Summary

You learned:

- What labels are
- Why labels exist
- Label structure
- Label best practices
- Recommended Kubernetes labels
- What selectors are
- Equality-based selectors
- How to filter resources using labels
- Basic label YAML

You now understand how Kubernetes identifies and groups resources using labels.

# K8S-15 - Labels, Selectors & Annotations (Part 2)

> **Class Date:** 07-Jul-2026
>
> **File:** `K8S-15_Labels_Selectors_Annotations_07-Jul-2026.md`

---

# 📚 Table of Contents

10. Set-Based Selectors
11. matchLabels
12. matchExpressions
13. Selector Operators
14. Deployment Selectors
15. Service Selectors
16. ReplicaSet Selectors
17. Selector Immutability
18. Labels vs Annotations
19. Production YAML
20. Part 2 Summary

---

# 10. Set-Based Selectors

Equality selectors are useful for simple matching.

Sometimes you need more flexibility.

Example:

Instead of:

```
environment=production
```

You want:

```
production

OR

staging
```

Set-based selectors provide this capability.

---

## Example

```yaml
matchExpressions:

- key: environment

  operator: In

  values:

  - production

  - staging
```

This matches:

```
production

OR

staging
```

---

# 11. matchLabels

The simplest selector syntax inside Kubernetes resources.

Example:

```yaml
selector:

  matchLabels:

    app: frontend

    environment: production
```

Meaning:

```
app=frontend

AND

environment=production
```

Both labels must match.

---

## Architecture

```
Pods

↓

app=frontend

environment=production

↓

matchLabels

↓

Selected
```

---

# 12. matchExpressions

Provides advanced filtering.

Example:

```yaml
selector:

  matchExpressions:

  - key: environment

    operator: In

    values:

    - production

    - staging
```

---

Multiple expressions are combined with logical AND.

Example:

```yaml
matchExpressions:

- key: environment

  operator: In

  values:

  - production

  - staging

- key: tier

  operator: In

  values:

  - web
```

Matches only Pods that satisfy **both** conditions.

---

# 13. Selector Operators

---

## In

```yaml
operator: In
```

Matches if the label value is in the list.

Example:

```
production

staging
```

---

## NotIn

```yaml
operator: NotIn
```

Matches every value except those listed.

---

## Exists

```yaml
operator: Exists
```

Only checks whether the key exists.

Example:

```
team=platform

↓

Match
```

---

## DoesNotExist

```yaml
operator: DoesNotExist
```

Matches resources that do not contain the specified label key.

---

# Operator Summary

| Operator | Description |
|----------|-------------|
| In | Value must exist in list |
| NotIn | Value must not exist in list |
| Exists | Key must exist |
| DoesNotExist | Key must be absent |

---

# 14. Deployment Selectors

Deployments identify the Pods they manage using selectors.

Example:

```yaml
spec:

  selector:

    matchLabels:

      app: frontend
```

Pod template:

```yaml
template:

  metadata:

    labels:

      app: frontend
```

These labels **must match**.

---

## Architecture

```
Deployment

↓

Selector

↓

app=frontend

↓

Pods

↓

app=frontend
```

---

If they don't match:

```
Deployment

↓

Cannot Manage Pods
```

---

# 15. Service Selectors

A Service discovers backend Pods using selectors.

Example:

```yaml
selector:

  app: payment
```

Pods:

```
payment-1

payment-2

payment-3
```

Only matching Pods receive traffic.

---

## Architecture

```
Client

↓

Service

↓

Selector

↓

payment Pods

↓

Responses
```

---

# 16. ReplicaSet Selectors

ReplicaSets use selectors to determine which Pods belong to them.

Example:

```yaml
selector:

  matchLabels:

    app: nginx
```

The Pod template must include the same labels.

---

# 17. Selector Immutability

One of the most important interview topics.

For Deployments, the selector is generally **immutable** after creation.

Example:

Created with:

```yaml
selector:

  matchLabels:

    app: payment
```

Later changing it to:

```yaml
app: frontend
```

is not supported because it would fundamentally change which Pods the Deployment manages.

Plan your selectors carefully before creating the resource.

---

# 18. Labels vs Annotations

Both are metadata, but they serve different purposes.

| Labels | Annotations |
|---------|-------------|
| Used for selection | Not used for selection |
| Small identifying metadata | Arbitrary metadata |
| Queried by selectors | Ignored by selectors |
| Drive controller behavior | Store additional information |

---

## Labels Example

```yaml
labels:

  app: payment

  version: v2
```

---

## Annotations Example

```yaml
annotations:

  description: Payment API

  owner: Platform Team

  documentation: https://internal.example/docs
```

Annotations can store:

- Build information
- Git commit SHA
- Deployment notes
- Contact details
- Tool-specific configuration

---

# 19. Production YAML

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: payment

  labels:

    app.kubernetes.io/name: payment

    app.kubernetes.io/component: backend

spec:

  replicas: 3

  selector:

    matchLabels:

      app: payment

  template:

    metadata:

      labels:

        app: payment

        environment: production

        version: v2

      annotations:

        git-commit: "a1b2c3d"

        owner: platform-team

    spec:

      containers:

      - name: payment

        image: payment:v2
```

---

## YAML Explanation

### metadata.labels

Identify the Deployment itself.

---

### selector

Determines which Pods the Deployment manages.

---

### template.labels

Labels applied to newly created Pods.

---

### annotations

Stores additional metadata that controllers do not use for selection.

---

# 20. Part 2 Summary

You learned:

- Set-based selectors
- matchLabels
- matchExpressions
- Selector operators
- Deployment selectors
- Service selectors
- ReplicaSet selectors
- Selector immutability
- Labels vs annotations
- Production Deployment YAML

You now understand how Kubernetes controllers locate, group, and manage resources using labels and selectors.
# K8S-15 - Labels, Selectors & Annotations (Part 3)

> **Class Date:** 07-Jul-2026

---

# 📚 Table of Contents

21. Labeling Strategies
22. Namespace Labels
23. Node Labels
24. Scheduling with Labels
25. Common Mistakes
26. Troubleshooting
27. Production Best Practices
28. Hands-on Labs
29. CKA Exam Tips
30. Interview Questions
31. Cheat Sheet
32. Chapter Summary

---

# 21. Labeling Strategies

A consistent labeling strategy makes Kubernetes clusters easier to operate.

A common production pattern is:

```yaml
labels:
  app.kubernetes.io/name: payment
  app.kubernetes.io/instance: payment-prod
  app.kubernetes.io/version: "2.1.0"
  app.kubernetes.io/component: backend
  app.kubernetes.io/part-of: ecommerce
  app.kubernetes.io/managed-by: helm

  environment: production
  team: payments
  cost-center: engineering
```

Benefits:

- Easier monitoring
- Better GitOps organization
- Cost allocation
- Resource ownership
- Easier troubleshooting

---

# 22. Namespace Labels

Namespaces themselves can have labels.

Example:

```yaml
apiVersion: v1
kind: Namespace

metadata:
  name: production

  labels:
    environment: production
    owner: platform-team
```

Namespace labels are useful for:

- Policy engines
- Admission controllers
- Reporting
- Resource organization

Example:

```bash
kubectl get namespaces --show-labels
```

---

# 23. Node Labels

Every node contains labels.

View them:

```bash
kubectl get nodes --show-labels
```

Example:

```
kubernetes.io/os=linux

kubernetes.io/arch=amd64

topology.kubernetes.io/zone=ap-south-1a
```

Cloud providers and Kubernetes automatically apply many standard node labels.

You can also add custom labels.

Example:

```bash
kubectl label node worker-1 gpu=true
```

---

# 24. Scheduling with Labels

Pods can target labeled nodes.

Example:

```yaml
spec:

  nodeSelector:

    gpu: "true"
```

Architecture:

```
Nodes

├── worker-1 (gpu=true)

├── worker-2 (gpu=false)

↓

Pod

↓

nodeSelector

↓

worker-1
```

For more advanced scheduling, use **Node Affinity**, which supports expressive matching with `matchExpressions`.

---

# 25. Common Mistakes

## Selector Does Not Match Labels

Deployment:

```yaml
selector:

  matchLabels:

    app: payment
```

Pod:

```yaml
labels:

  app: frontend
```

Result:

```
Deployment

↓

No Matching Pods
```

---

## Inconsistent Label Keys

Avoid mixing:

```
app

application

service-name
```

Choose a standard and use it consistently.

---

## Using Labels for Large Text

Avoid:

```yaml
labels:

  description: >
    This is a very long application description...
```

Use annotations instead.

---

## Frequently Changing Labels

Avoid labels that change on every deployment, such as timestamps.

Prefer annotations for dynamic metadata.

---

# 26. Troubleshooting

## Show Labels

```bash
kubectl get pods --show-labels
```

---

## Describe Resource

```bash
kubectl describe pod payment-123
```

Inspect:

```
Labels:
Annotations:
```

---

## Filter by Label

```bash
kubectl get pods -l app=payment
```

---

## Multiple Labels

```bash
kubectl get pods \
  -l app=payment,environment=production
```

---

## Set-Based Query

```bash
kubectl get pods \
  -l 'environment in (production,staging)'
```

---

## Check Deployment Selector

```bash
kubectl describe deployment payment
```

Verify:

```
Selector:
```

matches the Pod template labels.

---

# Common Problems

| Problem | Possible Cause | Solution |
|----------|----------------|----------|
| Service has no endpoints | Selector doesn't match Pod labels | Verify labels and selector |
| Deployment shows 0 Pods | Selector mismatch | Compare `spec.selector` with `template.metadata.labels` |
| Pod scheduled on wrong nodes | Incorrect node labels | Verify node labels and `nodeSelector`/affinity |
| Monitoring misses workloads | Required labels absent | Apply consistent labeling strategy |
| GitOps grouping incorrect | Missing recommended labels | Adopt `app.kubernetes.io/*` labels |

---

# 27. Production Best Practices

## Adopt Standard Labels

Use the Kubernetes recommended label schema wherever practical.

---

## Keep Labels Stable

Changing labels can affect:

- Services
- Deployments
- Network Policies
- Monitoring
- Automation

---

## Use Annotations for Metadata

Examples:

- Git commit SHA
- Build ID
- Documentation links
- Deployment notes

---

## Keep Selectors Simple

Use stable labels such as:

```text
app
component
environment
```

Avoid selecting on labels that change frequently.

---

## Document Your Label Strategy

A documented convention helps every team label resources consistently.

---

# 28. Hands-on Labs

## Lab 1

Create Pods with labels.

Verify:

```bash
kubectl get pods --show-labels
```

---

## Lab 2

Filter Pods:

```bash
kubectl get pods -l app=frontend
```

---

## Lab 3

Create a Service using:

```yaml
selector:
  app: frontend
```

Verify only matching Pods receive traffic.

---

## Lab 4

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

## Lab 5

Add annotations:

```yaml
annotations:
  git-commit: "abc123"
```

Verify:

```bash
kubectl describe pod
```

---

# 29. CKA Exam Tips

✔ Know the difference:

- Labels
- Selectors
- Annotations

✔ Memorize:

- `matchLabels`
- `matchExpressions`
- `In`
- `NotIn`
- `Exists`
- `DoesNotExist`

✔ Practice:

```bash
kubectl label
kubectl get --show-labels
kubectl get -l
kubectl describe
```

✔ Remember:

- Services discover Pods using selectors.
- Deployments manage Pods using selectors.
- Selectors should match Pod template labels.

---

# 30. Interview Questions

## Beginner

1. What is a label?
2. What is a selector?
3. What is an annotation?
4. Why are labels important?
5. How do Services use selectors?

---

## Intermediate

1. Explain `matchLabels`.
2. Explain `matchExpressions`.
3. What is selector immutability?
4. When should annotations be used?
5. How do labels help GitOps?

---

## Advanced

1. Design a labeling strategy for a large enterprise platform.
2. Explain how monitoring tools use labels.
3. How would you troubleshoot a Service with no endpoints?
4. Compare `nodeSelector` and Node Affinity.
5. Explain recommended Kubernetes labels.

---

## Scenario-Based

### Scenario 1

A Service has no endpoints.

What would you check first?

---

### Scenario 2

A Deployment creates no Pods.

Which labels and selectors would you verify?

---

### Scenario 3

Your organization has three environments: dev, staging, and production.

How would you design a labeling strategy?

---

### Scenario 4

A Pod must run only on GPU nodes.

Which Kubernetes feature would you use?

---

### Scenario 5

A GitOps dashboard groups applications incorrectly.

How could consistent labels help?

---

# 31. Labels & Selectors Cheat Sheet

Show labels:

```bash
kubectl get pods --show-labels
```

Filter by label:

```bash
kubectl get pods -l app=payment
```

Multiple labels:

```bash
kubectl get pods \
  -l app=payment,environment=production
```

Set-based selector:

```bash
kubectl get pods \
  -l 'environment in (production,staging)'
```

Add a label:

```bash
kubectl label pod payment-1 version=v2
```

Remove a label:

```bash
kubectl label pod payment-1 version-
```

Show node labels:

```bash
kubectl get nodes --show-labels
```

---

# 32. Chapter Summary

Congratulations! 🎉

You have completed **K8S-15 – Labels, Selectors & Annotations**.

In this chapter, you learned:

- Labels
- Recommended Kubernetes labels
- Equality-based selectors
- Set-based selectors
- `matchLabels`
- `matchExpressions`
- Selector operators
- Annotations
- Deployment selectors
- Service selectors
- Node labels
- Namespace labels
- Scheduling with labels
- Troubleshooting
- Best practices
- Hands-on labs
- CKA preparation
- Interview questions

You now understand one of Kubernetes' most fundamental concepts: how resources are identified, grouped, selected, and managed using labels and selectors.
