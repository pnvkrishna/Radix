# K8S-29 – GitOps (Argo CD & Flux) (Part 1)

> **Class Date:** 21-Jul-2026
>
> **Module:** GitOps for Kubernetes


---

# 📚 Table of Contents

1. Introduction to GitOps
2. Why GitOps?
3. Traditional CI/CD vs GitOps
4. GitOps Principles
5. Desired State vs Actual State
6. Git as the Single Source of Truth
7. GitOps Workflow
8. Git Repository Structure
9. GitOps Best Practices
10. Part 1 Summary

---

# 1. Introduction to GitOps

GitOps is an operational model where **Git becomes the single source of truth** for infrastructure and application deployment.

Instead of manually applying changes:

```
Developer

↓

kubectl apply

↓

Cluster
```

GitOps works like this:

```
Developer

↓

Git Commit

↓

Git Repository

↓

GitOps Controller

↓

Kubernetes Cluster
```

The cluster continuously reconciles itself with the desired configuration stored in Git.

---

# 2. Why GitOps?

Traditional deployments often introduce:

- Manual errors
- Configuration drift
- Limited auditability
- Difficult rollbacks
- Inconsistent environments

GitOps addresses these issues.

---

## Benefits

✅ Version control

✅ Audit history

✅ Easy rollback

✅ Declarative infrastructure

✅ Automated reconciliation

✅ Improved security

✅ Repeatable deployments

---

# 3. Traditional CI/CD vs GitOps

## Traditional Push Model

```
Developer

↓

CI Pipeline

↓

kubectl apply

↓

Cluster
```

Characteristics:

- CI system pushes changes
- CI requires cluster credentials
- Drift may go unnoticed

---

## GitOps Pull Model

```
Developer

↓

Git Repository

↓

GitOps Controller

↓

Cluster
```

Characteristics:

- Cluster pulls desired state
- Controller reconciles continuously
- Drift is detected and corrected

---

## Comparison

| Traditional CI/CD | GitOps |
|-------------------|---------|
| Push model | Pull model |
| CI accesses cluster | Cluster accesses Git |
| Drift possible | Drift continuously corrected |
| Rollback may require scripts | Rollback through Git history |
| Manual hotfixes common | Git remains the source of truth |

---

# 4. GitOps Principles

GitOps is based on four core ideas.

---

## Principle 1 — Declarative

Everything is described declaratively.

Examples:

- Deployments
- Services
- ConfigMaps
- RBAC
- NetworkPolicies

---

## Principle 2 — Version Controlled

Every change goes through Git.

```
Commit

↓

Pull Request

↓

Review

↓

Merge
```

---

## Principle 3 — Automatically Applied

A GitOps controller detects new commits and synchronizes the cluster.

---

## Principle 4 — Continuously Reconciled

The controller constantly compares:

```
Desired State (Git)

↓

Actual State (Cluster)
```

If they differ:

```
Reconcile

↓

Cluster Updated
```

---

# 5. Desired State vs Actual State

Desired State:

```
replicas: 3
image: nginx:1.30
```

Actual State:

```
replicas: 2
image: nginx:1.29
```

GitOps detects the drift and reconciles the cluster to match Git.

---

## Configuration Drift

```
Git

↓

Deployment

↓

Manual kubectl edit

↓

Cluster Changed

↓

Controller Detects Drift

↓

Restore Desired State
```

---

# 6. Git as the Single Source of Truth

Git should contain:

- Kubernetes manifests
- Helm charts
- Kustomize overlays
- RBAC policies
- NetworkPolicies
- Application configuration

Avoid storing sensitive secrets directly in Git unless using an appropriate encryption or secret-management solution (for example, Sealed Secrets, SOPS, or External Secrets).

---

## Benefits

- Traceability
- Peer review
- Rollback
- Compliance
- Change history

---

# 7. GitOps Workflow

```
Developer

↓

Feature Branch

↓

Pull Request

↓

Code Review

↓

Merge

↓

Git Repository

↓

GitOps Controller

↓

Kubernetes Cluster

↓

Application Running
```

---

## Deployment Flow

```
Git Commit

↓

Webhook / Polling

↓

Controller Detects Change

↓

Sync

↓

Health Check

↓

Application Ready
```

---

# 8. Git Repository Structure

Example:

```
gitops-repo/

├── apps/
│   ├── payment/
│   ├── orders/
│   └── inventory/
│
├── infrastructure/
│   ├── ingress/
│   ├── monitoring/
│   ├── cert-manager/
│   └── networking/
│
├── clusters/
│   ├── dev/
│   ├── test/
│   ├── staging/
│   └── production/
│
└── README.md
```

---

## Environment Separation

```
clusters/

↓

dev

↓

staging

↓

production
```

Each environment can have different:

- Replica counts
- Resource limits
- Image tags
- Domain names
- Feature flags

---

# 9. GitOps Best Practices

## Keep Everything Declarative

Avoid manual production changes.

---

## Protect Main Branch

Require:

- Pull Requests
- Reviews
- CI validation

---

## Use Small Commits

Smaller changes simplify:

- Reviews
- Rollbacks
- Troubleshooting

---

## Separate Infrastructure and Applications

Example:

```
Infrastructure Repository

↓

Cluster Components

Application Repository

↓

Business Applications
```

---

## Monitor Drift

GitOps controllers should continuously reconcile cluster state.

Investigate repeated drift instead of simply allowing endless reconciliation—it may indicate manual changes or an external controller modifying resources.

---

# GitOps Maturity Model

```
Level 1

Git Stores YAML

↓

Level 2

Automated Deployment

↓

Level 3

Continuous Reconciliation

↓

Level 4

Self-Healing Platform
```

---

# 10. Part 1 Summary

You learned:

- GitOps fundamentals
- Why GitOps matters
- Traditional CI/CD vs GitOps
- GitOps principles
- Desired vs Actual state
- Configuration drift
- Git as the source of truth
- GitOps workflow
- Repository organization
- GitOps best practices

You now understand the concepts that make GitOps the preferred deployment model for modern Kubernetes platforms.

# K8S-29 – GitOps (Argo CD & Flux) (Part 2)

> **Class Date:** 21-Jul-2026
>
> **Module:** GitOps with Argo CD

---

# 📚 Table of Contents

11. What is Argo CD?
12. Argo CD Architecture
13. Installing Argo CD
14. Accessing Argo CD
15. Applications
16. AppProjects
17. Sync Policies
18. Self-Heal & Prune
19. Health & Sync Status
20. Production Workflow
21. Part 2 Summary

---

# 11. What is Argo CD?

Argo CD is a **GitOps continuous delivery controller** for Kubernetes.

It continuously monitors Git repositories and reconciles the cluster so that the actual state matches the desired state stored in Git.

```
Developer

↓

Git Repository

↓

Argo CD

↓

Kubernetes Cluster
```

---

## Key Features

- Declarative deployments
- Continuous reconciliation
- Automatic drift detection
- Rollback using Git history
- Multi-cluster support
- Web UI
- CLI
- RBAC integration
- Notifications (with optional integrations)

---

# 12. Argo CD Architecture

```
               Git Repository
                     │
                     ▼
              Repository Server
                     │
                     ▼
              Application Controller
                     │
         ┌───────────┴───────────┐
         ▼                       ▼
     Kubernetes API         Argo CD API Server
                                     │
                                     ▼
                                 Web UI / CLI
```

---

## Core Components

| Component | Purpose |
|-----------|----------|
| API Server | UI, CLI, API access |
| Repository Server | Reads Git repositories |
| Application Controller | Reconciliation engine |
| Redis | Caching and performance |

---

# 13. Installing Argo CD

Create the namespace:

```bash
kubectl create namespace argocd
```

Install Argo CD:

```bash
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

---

## Verify Installation

```bash
kubectl get pods -n argocd
```

Expected components include:

- argocd-server
- argocd-repo-server
- argocd-application-controller
- argocd-redis

---

# 14. Accessing Argo CD

Port-forward (for local testing):

```bash
kubectl port-forward svc/argocd-server \
  -n argocd 8080:443
```

Access:

```
https://localhost:8080
```

---

## Initial Password

Retrieve the initial admin password:

```bash
kubectl -n argocd \
get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
```

> Change or disable the default admin account after initial setup in production.

---

# 15. Applications

An **Application** is the main Argo CD resource that defines:

- Git repository
- Path
- Destination cluster
- Namespace
- Sync policy

---

## Application Flow

```
Git Repository

↓

Application

↓

Sync

↓

Cluster
```

---

## Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: payment-app

spec:
  project: default

  source:
    repoURL: https://github.com/company/gitops
    path: apps/payment
    targetRevision: main

  destination:
    server: https://kubernetes.default.svc
    namespace: payment

  syncPolicy:
    automated: {}
```

---

### YAML Explanation

```yaml
project: default
```

Application belongs to the **default AppProject**.

---

```yaml
repoURL:
```

Git repository containing manifests.

---

```yaml
path:
```

Directory inside the repository.

---

```yaml
targetRevision:
```

Git branch, tag, or commit.

---

```yaml
destination:
```

Target cluster and namespace.

---

```yaml
syncPolicy:
```

Defines synchronization behavior.

---

# 16. AppProjects

AppProjects group Applications and enforce security boundaries.

Example capabilities:

- Allowed repositories
- Allowed namespaces
- Allowed clusters
- Allowed resource kinds

---

## Example

```
Finance Project

↓

Finance Apps

↓

Finance Namespaces
```

Benefits:

- Multi-team isolation
- Governance
- RBAC integration

---

# 17. Sync Policies

## Manual Sync

```
Git Change

↓

OutOfSync

↓

Manual Approval

↓

Sync
```

Useful for production environments requiring controlled deployments.

---

## Automatic Sync

```
Git Commit

↓

Detected

↓

Automatically Applied
```

Configured as:

```yaml
syncPolicy:
  automated: {}
```

---

# 18. Self-Heal & Prune

## Self-Heal

If someone changes a resource manually:

```
Git

↓

Desired State

↓

Manual Change

↓

Argo CD Detects Drift

↓

Restore Desired State
```

Enable:

```yaml
syncPolicy:
  automated:
    selfHeal: true
```

---

## Prune

Removes resources deleted from Git.

```
Git Delete

↓

Sync

↓

Resource Removed
```

Enable:

```yaml
syncPolicy:
  automated:
    prune: true
```

> Use pruning carefully in production. Deleting a manifest from Git may remove the corresponding resource from the cluster.

---

# 19. Health & Sync Status

Argo CD reports two important states.

---

## Sync Status

| Status | Meaning |
|--------|----------|
| Synced | Cluster matches Git |
| OutOfSync | Drift detected |
| Unknown | Unable to determine state |

---

## Health Status

| Status | Meaning |
|--------|----------|
| Healthy | Resource operating normally |
| Progressing | Deployment in progress |
| Degraded | Resource unhealthy |
| Missing | Resource not found |
| Suspended | Resource intentionally paused (where applicable) |

---

# 20. Production Workflow

```
Developer

↓

Feature Branch

↓

Pull Request

↓

Review

↓

Merge

↓

Git Repository

↓

Argo CD Detects Change

↓

Sync

↓

Health Check

↓

Application Available
```

---

## Production Best Practices

### Protect Git

- Branch protection
- Pull requests
- Required reviews
- CI validation

---

### Avoid Manual Changes

Treat the cluster as **read-only** for application configuration.

All desired changes should originate from Git.

---

### Use AppProjects

Separate:

- Development
- Testing
- Production
- Teams

---

### Monitor Sync Failures

Investigate:

- Authentication errors
- Git access failures
- Invalid manifests
- Health check failures

---

# 21. Part 2 Summary

You learned:

- Argo CD architecture
- Installation
- Access methods
- Applications
- AppProjects
- Manual and automatic sync
- Self-heal
- Prune
- Health status
- Sync status
- Production GitOps workflow

You now understand how Argo CD continuously deploys and reconciles Kubernetes workloads from Git.


# K8S-29 – GitOps (Argo CD & Flux) (Part 3)

> **Class Date:** 21-Jul-2026
>
> **Module:** GitOps for Kubernetes

---

# 📚 Table of Contents

22. What is Flux?
23. Flux Architecture
24. Installing Flux
25. Flux Reconciliation
26. HelmRelease & Kustomization
27. Argo CD vs Flux
28. Multi-Cluster GitOps
29. Progressive Delivery Concepts
30. GitOps Security
31. Production GitOps Architecture
32. Hands-on Labs
33. CKA / CKAD / CKS Interview Questions
34. GitOps Cheat Sheet
35. Chapter Summary

---

# 22. What is Flux?

Flux is a CNCF GitOps toolkit that continuously reconciles Kubernetes clusters with the desired state stored in Git.

```
Git Repository

↓

Flux Controllers

↓

Kubernetes Cluster
```

Unlike Argo CD, Flux is built as a collection of specialized controllers rather than a single UI-centric platform.

---

## Key Features

- Pull-based GitOps
- Git reconciliation
- Helm support
- Kustomize support
- OCI artifact support
- Image automation
- Multi-cluster deployments

---

# 23. Flux Architecture

```
                Git Repository
                       │
                       ▼
                Source Controller
                       │
        ┌──────────────┴──────────────┐
        ▼                             ▼
 Kustomize Controller         Helm Controller
        │                             │
        └──────────────┬──────────────┘
                       ▼
               Kubernetes Cluster
```

---

## Main Controllers

| Controller | Responsibility |
|------------|----------------|
| Source Controller | Watches Git, OCI, and Helm repositories |
| Kustomize Controller | Applies Kustomize resources |
| Helm Controller | Manages Helm releases |
| Notification Controller | Sends GitOps events |
| Image Automation Controllers | Update manifests with approved image changes |

---

# 24. Installing Flux

Install the Flux CLI.

Bootstrap a cluster:

```bash
flux bootstrap github \
  --owner=<github-user> \
  --repository=gitops-repo \
  --branch=main \
  --path=clusters/production
```

The bootstrap process typically:

- Installs Flux controllers
- Connects the cluster to Git
- Creates the initial GitOps configuration

---

## Verify

```bash
flux check
```

---

View resources:

```bash
flux get all
```

---

# 25. Flux Reconciliation

Flux continuously compares Git with the cluster.

```
Git Commit

↓

Source Controller

↓

Kustomize / Helm Controller

↓

Apply Changes

↓

Cluster Updated
```

If manual drift occurs:

```
Manual Change

↓

Detected

↓

Reconciled

↓

Desired State Restored
```

---

# 26. HelmRelease & Kustomization

Flux introduces Kubernetes Custom Resources for managing deployments.

---

## Kustomization

Example:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization

metadata:
  name: apps

spec:
  interval: 5m
  path: ./apps
  prune: true
  sourceRef:
    kind: GitRepository
    name: platform-config
```

### Key Fields

- `interval` – How often reconciliation runs.
- `path` – Directory in the source repository.
- `prune` – Removes resources deleted from Git.
- `sourceRef` – Points to the GitRepository resource.

---

## HelmRelease

Example:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease

metadata:
  name: payment

spec:
  interval: 10m

  chart:
    spec:
      chart: payment
      sourceRef:
        kind: HelmRepository
        name: company-charts

  values:
    replicaCount: 3
```

Flux manages Helm lifecycle declaratively through Kubernetes resources.

---

# 27. Argo CD vs Flux

| Feature | Argo CD | Flux |
|---------|----------|------|
| CNCF Project | Yes | Yes |
| Pull-based GitOps | Yes | Yes |
| Web UI | Built-in | Optional (third-party dashboards available) |
| CLI | Yes | Yes |
| Helm Support | Yes | Yes |
| Kustomize Support | Yes | Yes |
| Multi-cluster | Yes | Yes |
| Drift Detection | Yes | Yes |
| Image Automation | Available via Argo CD Image Updater (separate project) | Built-in controllers |

---

## When to Choose?

### Argo CD

Best when:

- Teams want a rich UI
- Visual synchronization status is important
- Manual approval workflows are common

---

### Flux

Best when:

- Kubernetes-native APIs are preferred
- Git-centric workflows dominate
- Image automation is a priority

Both are excellent production choices.

---

# 28. Multi-Cluster GitOps

Example repository:

```
gitops/

├── clusters/
│   ├── dev/
│   ├── test/
│   ├── staging/
│   └── production/
│
├── infrastructure/
│
└── applications/
```

Each cluster reconciles only its own configuration.

---

## Deployment Flow

```
Git

↓

Cluster A

Cluster B

Cluster C

↓

All Continuously Reconciled
```

---

# 29. Progressive Delivery Concepts

Progressive delivery reduces deployment risk.

---

## Rolling Update

```
Old Pods

↓

Replace Gradually

↓

New Pods
```

---

## Blue–Green

```
Blue Environment

↓

Green Environment

↓

Traffic Switch
```

---

## Canary

```
10%

↓

30%

↓

60%

↓

100%
```

Monitor metrics between stages before increasing traffic.

> Tools such as **Argo Rollouts** or **Flagger** can automate progressive delivery workflows.

---

# 30. GitOps Security

## Protect Git

- Branch protection
- Signed commits (where adopted)
- Required reviews
- CI validation

---

## Protect Controllers

- RBAC
- Namespace isolation
- Least privilege

---

## Secrets

Avoid plaintext secrets in Git.

Prefer:

- Sealed Secrets
- SOPS
- External Secrets Operator

---

## Audit

Track:

- Git commits
- Pull requests
- Sync events
- Deployment history

---

# 31. Production GitOps Architecture

```
                 Developers
                      │
                      ▼
                Git Repository
                      │
          Pull Request + Review
                      │
                      ▼
              GitOps Controller
          (Argo CD or Flux)
                      │
        ┌─────────────┴─────────────┐
        ▼                           ▼
   Dev Cluster                Production Cluster
        │                           │
        ▼                           ▼
   Health Checks              Continuous Reconciliation
```

---

## Production Best Practices

- Git is the source of truth
- Protect production branches
- Use separate environments
- Monitor reconciliation failures
- Encrypt secrets
- Review every production change
- Keep Git history clean and meaningful

---

# 32. Hands-on Labs

## Lab 1

Bootstrap Flux into a test cluster.

Verify all controllers are healthy.

---

## Lab 2

Deploy an application using a Flux `Kustomization`.

Observe automatic reconciliation after a Git change.

---

## Lab 3

Deploy a Helm chart using a `HelmRelease`.

Modify values and verify reconciliation.

---

## Lab 4

Create an Argo CD Application.

Enable:

- Automatic Sync
- Self-Heal
- Prune

Observe reconciliation after a controlled configuration change.

---

## Lab 5

Design a GitOps repository supporting:

- Development
- Staging
- Production

Include separate application and infrastructure directories.

---

# 33. CKA / CKAD / CKS Interview Questions

> **Note:** GitOps tools themselves are not primary exam objectives for CKA, CKAD, or CKS, but GitOps concepts and declarative operations are highly relevant in production Kubernetes interviews.

## Beginner

1. What is GitOps?
2. Why is Git the source of truth?
3. What is reconciliation?
4. What is configuration drift?
5. Compare push vs pull deployment models.

---

## Intermediate

1. Explain Argo CD architecture.
2. Explain Flux controllers.
3. What is a Flux `Kustomization`?
4. What is an Argo CD `Application`?
5. Compare Argo CD and Flux.

---

## Advanced

1. Design a multi-cluster GitOps platform.
2. How would you secure GitOps?
3. How would you manage secrets?
4. Explain progressive delivery with GitOps.
5. How would you recover from accidental configuration drift?

---

## Scenario-Based

### Scenario 1

A manual production change is immediately reverted.

Why?

---

### Scenario 2

Argo CD reports `OutOfSync`, but the application appears healthy.

What would you investigate?

---

### Scenario 3

Flux reconciliation fails after a Git commit.

Which logs and resources would you inspect?

---

### Scenario 4

A Helm release repeatedly fails to reconcile.

How would you isolate the root cause?

---

### Scenario 5

You need to deploy the same application to five clusters with environment-specific configuration.

How would you organize the Git repository?

---

# 34. GitOps Cheat Sheet

## Argo CD

Install:

```bash
kubectl create namespace argocd

kubectl apply -n argocd \
-f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

List applications:

```bash
argocd app list
```

Sync:

```bash
argocd app sync <app-name>
```

---

## Flux

Bootstrap:

```bash
flux bootstrap github ...
```

Health:

```bash
flux check
```

Resources:

```bash
flux get all
```

Reconcile manually:

```bash
flux reconcile kustomization <name>
```

---

## GitOps Principles

- Git is the source of truth
- Declarative configuration
- Pull-based reconciliation
- Continuous drift correction
- Automated deployments

---

# 35. Chapter Summary

Congratulations! 🎉

You have completed **K8S-29 – GitOps (Argo CD & Flux)**.

In this chapter, you learned:

- GitOps fundamentals
- Desired vs actual state
- Git reconciliation
- Argo CD architecture
- Applications
- AppProjects
- Sync policies
- Self-heal and prune
- Flux architecture
- Flux controllers
- HelmRelease
- Kustomization
- Argo CD vs Flux
- Multi-cluster GitOps
- Progressive delivery concepts
- GitOps security
- Production architectures
- Hands-on labs
- Interview questions

You now have a production-ready understanding of GitOps workflows and can design, deploy, and operate Kubernetes environments using either Argo CD or Flux.

