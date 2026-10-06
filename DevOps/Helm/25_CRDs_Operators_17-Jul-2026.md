# K8S-25 - Custom Resource Definitions (CRDs) & Operators (Part 1)

> **Class Date:** 17-Jul-2026
>
> **Module:** Kubernetes API Extensions
>
> **File Name:** `K8S-25_CRDs_Operators_17-Jul-2026.md`

---

# 📚 Table of Contents

1. Introduction
2. Why Extend Kubernetes?
3. Kubernetes API Architecture
4. What is a CRD?
5. Custom Resources (CRs)
6. Built-in Resources vs Custom Resources
7. First CRD Example
8. CRD Validation Schema
9. Part 1 Summary

---

# 1. Introduction

Kubernetes comes with built-in resources such as:

- Pods
- Deployments
- Services
- ConfigMaps
- Secrets
- StatefulSets

But what if you want Kubernetes to manage:

- MySQL databases
- Kafka clusters
- Redis clusters
- Certificates
- External secrets
- Cloud databases
- AI model deployments

Kubernetes allows you to create **your own API resources**.

These are called **Custom Resource Definitions (CRDs).**

---

# 2. Why Extend Kubernetes?

Imagine Kubernetes only understood:

```
Pod
Service
Deployment
```

How would you represent:

```
Database
```

or

```
Kafka Cluster
```

or

```
Certificate
```

You can't with only the built-in resources.

CRDs solve this problem.

---

# Example

Instead of:

```
Deployment

↓

Database
```

You can create:

```
MySQL

↓

Cluster
```

as a native Kubernetes resource.

---

# 3. Kubernetes API Architecture

```
kubectl

        │

        ▼

API Server

        │

 ┌──────┴──────┐

 ▼             ▼

Built-in APIs  Custom APIs (CRDs)
```

The API Server stores both built-in resources and custom resources.

---

# 4. What is a CRD?

A **Custom Resource Definition** extends the Kubernetes API by defining a new resource type.

Example:

```
kubectl get mysqlclusters
```

This command works only after the corresponding CRD is installed.

---

# CRD Analogy

Think of Kubernetes as an operating system.

Built-in resources are like built-in file types.

CRDs let you define **new file types** that Kubernetes understands.

---

# 5. Custom Resources (CRs)

Once a CRD exists, users create **Custom Resources (CRs)**.

```
CRD

↓

Defines Resource Type

↓

Custom Resource

↓

Actual Instance
```

Example:

CRD:

```
MySQLCluster
```

Custom Resource:

```yaml
apiVersion: database.example.com/v1
kind: MySQLCluster

metadata:
  name: production-db

spec:
  replicas: 3
```

---

# 6. Built-in Resources vs Custom Resources

| Built-in Resource | Custom Resource |
|-------------------|-----------------|
| Pod | MySQLCluster |
| Deployment | KafkaCluster |
| Service | RedisCluster |
| Secret | ExternalSecret |
| Ingress | Certificate |

---

# Real-World Examples

Many popular Kubernetes projects install CRDs.

| Project | Custom Resource |
|----------|-----------------|
| Cert-Manager | Certificate |
| External Secrets Operator | ExternalSecret |
| Prometheus Operator | Prometheus |
| Strimzi | Kafka |
| Crossplane | Composite resources & managed cloud resources |

---

# 7. First CRD Example

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition

metadata:
  name: widgets.example.com

spec:

  group: example.com

  names:

    kind: Widget

    plural: widgets

    singular: widget

  scope: Namespaced

  versions:

  - name: v1

    served: true

    storage: true

    schema:

      openAPIV3Schema:

        type: object

        properties:

          spec:

            type: object
```

---

# YAML Explained

## group

```
example.com
```

API group.

---

## kind

```
Widget
```

Object type.

---

## plural

```
widgets
```

Used by:

```bash
kubectl get widgets
```

---

## scope

```
Namespaced
```

The resource exists inside a namespace.

Cluster-scoped CRDs are also possible.

---

# 8. CRD Validation Schema

CRDs can validate user input using an OpenAPI v3 schema.

Example:

```yaml
properties:

  spec:

    type: object

    properties:

      replicas:

        type: integer

      image:

        type: string
```

Now Kubernetes validates:

```yaml
spec:

  replicas: 3

  image: nginx
```

before storing the resource.

---

## Benefits

- Prevent invalid configurations
- Improve API consistency
- Catch errors early

---

# Verify

List CRDs:

```bash
kubectl get crd
```

Describe a CRD:

```bash
kubectl describe crd widgets.example.com
```

View API resources:

```bash
kubectl api-resources
```

---

# 9. Part 1 Summary

You learned:

- Why Kubernetes is extensible
- Kubernetes API architecture
- CRDs
- Custom Resources
- Built-in vs Custom resources
- First CRD example
- OpenAPI validation schema

You now understand how Kubernetes can be extended with entirely new API objects.

# K8S-25 – Custom Resource Definitions (CRDs) & Operators (Part 2)

> **Class Date:** 17-Jul-2026
>
> **Module:** Kubernetes API Extensions

---

# 📚 Table of Contents

10. Why Operators?
11. What is a Controller?
12. Reconciliation Loop
13. Desired State vs Actual State
14. Operator Architecture
15. Status & Scale Subresources
16. CRD Versioning
17. Operator SDK & Kubebuilder
18. Real-World Operators
19. Part 2 Summary

---

# 10. Why Operators?

Managing stateful applications manually is difficult.

Example:

```
MySQL Cluster

↓

Primary

↓

Replica 1

↓

Replica 2
```

If the primary database fails:

Without an Operator:

```
Administrator

↓

Detect Failure

↓

Promote Replica

↓

Update Clients

↓

Restore Replication
```

With an Operator:

```
Failure

↓

Operator Detects

↓

Promotes Replica

↓

Updates State

↓

Cluster Healthy
```

The Operator automates operational knowledge.

---

# 11. What is a Controller?

A **controller** continuously watches Kubernetes resources and works to make the actual cluster state match the desired state.

Example:

Deployment:

```yaml
replicas: 3
```

Current state:

```
2 Pods Running
```

Deployment controller:

```
Creates

↓

Third Pod
```

Controllers are already built into Kubernetes.

Examples:

- Deployment Controller
- ReplicaSet Controller
- Job Controller
- StatefulSet Controller
- Node Controller

An Operator is essentially a **custom controller** built for a custom resource.

---

# 12. Reconciliation Loop

The reconciliation loop is the core idea behind every controller and Operator.

```
Watch Resource

↓

Compare Desired State

↓

Compare Actual State

↓

Take Action

↓

Repeat
```

This loop runs continuously.

---

## Example

Desired:

```
Kafka Cluster

Replicas = 3
```

Actual:

```
2 Brokers Running
```

Operator:

```
Creates Broker

↓

3 Brokers Running
```

---

# 13. Desired State vs Actual State

Kubernetes is a **declarative system**.

You describe:

```
What You Want
```

not:

```
How To Do It
```

Example:

```yaml
spec:

  replicas: 5
```

Desired:

```
5 Pods
```

Actual:

```
4 Pods
```

Controller:

```
Creates One More Pod
```

Operators apply the same principle to custom resources.

---

# 14. Operator Architecture

```
Custom Resource

        │

        ▼

API Server

        │

        ▼

Operator Controller

        │

        ▼

External Systems / Kubernetes Resources

        │

        ▼

Status Updated
```

The Operator:

- Watches CRs
- Creates or updates resources
- Monitors health
- Updates the resource status

---

# Example Flow

```
MySQLCluster CR

↓

Operator

↓

StatefulSet

↓

Pods

↓

PersistentVolumes

↓

Services

↓

Status
```

Users interact only with the custom resource.

The Operator manages everything else.

---

# 15. Status & Scale Subresources

Custom Resources typically contain two important sections.

```
spec

↓

Desired State

----------------

status

↓

Observed State
```

---

## Example

```yaml
apiVersion: database.example.com/v1
kind: MySQLCluster

metadata:
  name: production-db

spec:

  replicas: 3

status:

  readyReplicas: 3

  phase: Running
```

---

## Why Keep Them Separate?

`spec`

- Written by users
- Declares intent

`status`

- Written by the controller
- Reports observations

This separation prevents users from directly modifying runtime status.

---

## Scale Subresource

A CRD can expose a **scale subresource**, allowing Kubernetes tools such as the Horizontal Pod Autoscaler (HPA) to interact with the custom resource's replica count, provided the Operator implements the necessary behavior.

---

# 16. CRD Versioning

CRDs support multiple API versions.

Example:

```
v1alpha1

↓

v1beta1

↓

v1
```

---

## Typical Lifecycle

| Version | Meaning |
|----------|---------|
| v1alpha1 | Experimental |
| v1beta1 | Feature complete, still evolving |
| v1 | Stable |

---

## Example

```yaml
versions:

- name: v1alpha1

- name: v1beta1

- name: v1
```

One version is marked as the storage version used by the API server.

---

# 17. Operator SDK & Kubebuilder

Two popular frameworks for building Operators in Go.

---

## Operator SDK

Provides scaffolding, packaging, and testing tools to simplify Operator development.

---

## Kubebuilder

A framework maintained under the Kubernetes ecosystem for building Kubernetes APIs and controllers using `controller-runtime`.

---

## Comparison

| Tool | Focus |
|------|-------|
| Kubebuilder | Kubernetes API & controller development |
| Operator SDK | Operator-focused tooling built on top of Kubernetes controller libraries |

Both commonly use the `controller-runtime` library.

---

# 18. Real-World Operators

---

## Prometheus Operator

Custom Resource:

```
Prometheus
```

Automatically manages:

- StatefulSets
- Services
- Monitoring configuration

---

## Cert-Manager

Custom Resource:

```
Certificate
```

Automatically:

- Requests certificates
- Renews certificates
- Updates Kubernetes Secrets

---

## External Secrets Operator

Custom Resource:

```
ExternalSecret
```

Synchronizes secrets from external secret stores into Kubernetes Secrets.

---

## Strimzi

Custom Resource:

```
Kafka
```

Automatically manages:

- Brokers
- ZooKeeper (older Kafka deployments)
- Kafka configuration
- Rolling updates

> Modern Kafka deployments using KRaft mode no longer require ZooKeeper.

---

## Crossplane

Custom Resources represent cloud infrastructure such as:

- Databases
- Buckets
- Virtual networks

Crossplane reconciles these resources with cloud providers.

---

# Operator Ecosystem

```
Developer

↓

Custom Resource

↓

Operator

↓

Automation

↓

Healthy System
```

---

# 19. Part 2 Summary

You learned:

- Why Operators exist
- Controllers
- Reconciliation loops
- Desired vs actual state
- Operator architecture
- `spec` vs `status`
- Scale subresources
- CRD versioning
- Operator SDK
- Kubebuilder
- Real-world Operators

You now understand how Kubernetes automates complex application lifecycle management through Operators.

# K8S-25 – Custom Resource Definitions (CRDs) & Operators (Part 3)

> **Class Date:** 17-Jul-2026
>
> **Module:** Kubernetes API Extensions

---

# 📚 Table of Contents

20. Creating a Simple CRD
21. Creating a Custom Resource
22. Installing Operators
23. Operator Lifecycle Manager (OLM)
24. Troubleshooting Operators
25. Common Mistakes
26. Production Best Practices
27. Hands-on Labs
28. CKA / CKAD / CKS Interview Questions
29. CRD & Operator Cheat Sheet
30. Real Production Case Studies
31. Chapter Summary

---

# 20. Creating a Simple CRD

A CRD defines a new API resource.

Example:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition

metadata:
  name: widgets.example.com

spec:
  group: example.com

  names:
    plural: widgets
    singular: widget
    kind: Widget

  scope: Namespaced

  versions:
  - name: v1

    served: true
    storage: true

    schema:
      openAPIV3Schema:
        type: object
```

Apply:

```bash
kubectl apply -f crd.yaml
```

Verify:

```bash
kubectl get crd
```

---

# 21. Creating a Custom Resource

Once the CRD exists, create an instance.

```yaml
apiVersion: example.com/v1
kind: Widget

metadata:
  name: demo-widget

spec:
  replicas: 3
```

Apply:

```bash
kubectl apply -f widget.yaml
```

Verify:

```bash
kubectl get widgets
```

Describe:

```bash
kubectl describe widget demo-widget
```

At this point, Kubernetes stores the object, but **nothing will act on it until a controller or Operator watches it**.

---

# 22. Installing Operators

Most production Operators are installed using one of these approaches:

- Helm
- Vendor manifests (`kubectl apply`)
- Operator Lifecycle Manager (OLM)

---

## Example: Helm

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update

helm install cert-manager \
  jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace
```

Always follow the vendor's installation guide for required CRDs and configuration.

---

## Verify Installation

```bash
kubectl get pods -n cert-manager
```

```bash
kubectl get crd
```

---

# 23. Operator Lifecycle Manager (OLM)

OLM simplifies Operator installation, upgrades, and lifecycle management.

---

## Architecture

```
OperatorHub

        │

        ▼

Catalog Source

        │

        ▼

Subscription

        │

        ▼

Operator Installed

        │

        ▼

CRDs Available
```

---

## Core OLM Resources

| Resource | Purpose |
|----------|---------|
| CatalogSource | Available Operators |
| Subscription | Tracks and installs an Operator |
| InstallPlan | Upgrade/install plan |
| ClusterServiceVersion (CSV) | Installed Operator metadata |

---

## Benefits

- Automated upgrades
- Version management
- Dependency handling
- Centralized Operator installation

---

# 24. Troubleshooting Operators

---

## Check Pods

```bash
kubectl get pods -A
```

---

## View Logs

```bash
kubectl logs deployment/<operator-name> -n <namespace>
```

---

## Check Custom Resources

```bash
kubectl get <resource>
```

Example:

```bash
kubectl get certificates
```

---

## Describe the Custom Resource

```bash
kubectl describe <resource> <name>
```

Look for:

- Events
- Conditions
- Status fields

---

## Inspect Status

```bash
kubectl get <resource> <name> -o yaml
```

Focus on:

```yaml
status:
```

This often contains reconciliation progress or error details.

---

## Verify CRDs

```bash
kubectl get crd
```

If the CRD is missing, Kubernetes will not recognize the custom resource type.

---

# Common Reconciliation Issues

| Problem | Possible Cause | Resolution |
|----------|----------------|-----------|
| CR created, nothing happens | Operator not running | Check Operator Pods and logs |
| Resource stuck | Validation or reconciliation error | Inspect `status` and Events |
| Unknown resource | CRD missing | Install the CRD |
| Status never updates | Controller not reconciling | Check controller logs |

---

# 25. Common Mistakes

---

## Mistake 1

Installing a Custom Resource before installing its CRD.

Result:

```
error: no matches for kind ...
```

---

## Mistake 2

Installing the CRD but forgetting the Operator.

The object exists but nothing manages it.

---

## Mistake 3

Ignoring the `status` section.

Most reconciliation information is reported there.

---

## Mistake 4

Editing generated resources directly.

Example:

```
StatefulSet

↓

Manual Edit

↓

Operator

↓

Reconciles

↓

Manual Changes Lost
```

Modify the **Custom Resource**, not the resources generated by the Operator.

---

## Mistake 5

Skipping version compatibility checks.

Always verify that:

- Kubernetes version
- Operator version
- CRD version

are supported together.

---

# 26. Production Best Practices

---

## Use Stable APIs

Prefer stable (`v1`) CRDs for production when available.

---

## Backup CRDs

CRDs define the API.

Custom Resources contain business configuration.

Back up **both** during disaster recovery planning.

---

## Monitor Operators

Monitor:

- Pod health
- Restart count
- Logs
- Reconciliation failures
- Conditions

---

## Use GitOps

Store:

- CRDs
- Operator installation manifests
- Custom Resources

in Git for version control and reproducibility.

---

## Avoid Manual Resource Changes

Treat Operator-managed resources as read-only.

Update the Custom Resource instead.

---

# 27. Hands-on Labs

---

## Lab 1

Install a CRD.

Verify:

```bash
kubectl api-resources
```

---

## Lab 2

Create a Custom Resource.

Observe it with:

```bash
kubectl get
kubectl describe
```

---

## Lab 3

Install an Operator such as Cert-Manager or External Secrets Operator in a test cluster.

Verify:

- Pods
- CRDs
- Custom Resources

---

## Lab 4

Modify a Custom Resource.

Observe how the Operator reconciles the managed resources.

---

## Lab 5

Delete an Operator-managed Deployment.

Watch the Operator recreate it if that is part of its reconciliation logic.

---

# 28. CKA / CKAD / CKS Interview Questions

> **Note:** Deep Operator development is outside the CKA curriculum, but CRDs, controllers, and Operators are highly relevant in **CKAD**, platform engineering, and production Kubernetes interviews.

## Beginner

1. What is a CRD?
2. What is a Custom Resource?
3. What is an Operator?
4. Why are Operators useful?
5. What is reconciliation?

---

## Intermediate

1. Compare CRDs and built-in resources.
2. Explain `spec` vs `status`.
3. What is the reconciliation loop?
4. Why should you modify the Custom Resource instead of generated resources?
5. What is OLM?

---

## Advanced

1. Design an Operator for a database platform.
2. Explain CRD versioning strategies.
3. How would you troubleshoot a failing Operator?
4. Compare Helm and Operators.
5. How would you safely upgrade an Operator in production?

---

## Scenario-Based

### Scenario 1

A Custom Resource is accepted by the API server, but no Pods are created.

What would you investigate?

---

### Scenario 2

An Operator repeatedly reverts manual changes to a StatefulSet.

Why?

---

### Scenario 3

Your team wants Kubernetes to manage cloud databases declaratively.

Would a CRD alone be enough? Why or why not?

---

### Scenario 4

A CRD upgrade introduces a new API version.

How would you approach migration?

---

### Scenario 5

An Operator crashes after deployment.

Which Kubernetes resources and logs would you inspect first?

---

# 29. CRD & Operator Cheat Sheet

List CRDs:

```bash
kubectl get crd
```

View API resources:

```bash
kubectl api-resources
```

Describe a CRD:

```bash
kubectl describe crd <name>
```

List Custom Resources:

```bash
kubectl get <resource>
```

Describe a Custom Resource:

```bash
kubectl describe <resource> <name>
```

View status:

```bash
kubectl get <resource> <name> -o yaml
```

View Operator logs:

```bash
kubectl logs deployment/<operator-name> -n <namespace>
```

---

# 30. Real Production Case Studies

## Cert-Manager

```
Certificate CR

↓

Operator

↓

Certificate Request

↓

Secret Updated

↓

Automatic Renewal
```

---

## External Secrets Operator

```
ExternalSecret

↓

Operator

↓

Cloud Secret Manager

↓

Kubernetes Secret
```

Applications continue to consume native Kubernetes Secrets.

---

## Prometheus Operator

```
Prometheus CR

↓

Operator

↓

StatefulSet

↓

Services

↓

Monitoring Stack
```

---

## Crossplane

```
Managed Resource

↓

Operator

↓

Cloud Provider API

↓

Cloud Resource Created
```

Developers use Kubernetes manifests while Crossplane provisions infrastructure.

---

# 31. Chapter Summary

Congratulations! 🎉

You have completed **K8S-25 – Custom Resource Definitions (CRDs) & Operators**.

In this chapter, you learned:

- CRDs
- Custom Resources
- OpenAPI validation
- Controllers
- Reconciliation loops
- Desired vs actual state
- `spec` vs `status`
- CRD versioning
- Operator SDK
- Kubebuilder
- OLM
- Installing Operators
- Troubleshooting Operators
- Production best practices
- Hands-on labs
- Interview questions

You now understand how Kubernetes extends its API and automates complex operational workflows through the Operator pattern.

