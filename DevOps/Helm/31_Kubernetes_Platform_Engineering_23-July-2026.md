# K8S-31 – Multi-Tenancy & Governance (Part 1)

> **Class Date:** 23-Jul-2026
>
> **Module:** Kubernetes Platform Engineering

---

# 📚 Table of Contents

1. Introduction
2. What is Multi-Tenancy?
3. Why Multi-Tenancy?
4. Soft vs Hard Multi-Tenancy
5. Tenant Isolation Models
6. Namespace-Based Isolation
7. ResourceQuota
8. LimitRange
9. Labels & Governance
10. Part 1 Summary

---

# 1. Introduction

A single Kubernetes cluster often hosts workloads for multiple:

- Teams
- Projects
- Departments
- Customers
- Business units

Example:

```
Production Cluster

├── Finance Team
├── HR Team
├── DevOps Team
├── AI Team
└── E-Commerce Team
```

Each tenant should operate **independently** while sharing the same cluster.

---

# 2. What is Multi-Tenancy?

Multi-tenancy allows multiple independent tenants to safely share Kubernetes infrastructure.

A tenant may be:

- A team
- A department
- A customer
- A project
- An application group

---

## Goal

Provide:

- Isolation
- Security
- Fair resource usage
- Governance
- Operational efficiency

---

# 3. Why Multi-Tenancy?

Without isolation:

```
Team A

↓

Consumes All CPU

↓

Team B Impacted
```

Without governance:

```
Developer

↓

Deploys Privileged Pod

↓

Security Risk
```

Without quotas:

```
Infinite Pods

↓

Cluster Exhausted
```

---

# 4. Soft vs Hard Multi-Tenancy

## Soft Multi-Tenancy

All tenants share:

- Cluster
- Nodes
- Control plane

Isolation is primarily logical.

```
Cluster

├── Namespace A
├── Namespace B
└── Namespace C
```

Suitable for:

- Internal engineering teams
- Development environments
- Trusted users

---

## Hard Multi-Tenancy

Isolation extends beyond namespaces.

Possible approaches include:

- Dedicated clusters
- Dedicated node pools
- Strong workload isolation
- Separate cloud accounts or subscriptions (depending on the platform)

Suitable for:

- Regulated industries
- Untrusted tenants
- SaaS providers
- High-security environments

---

## Comparison

| Soft | Hard |
|------|------|
| Namespace isolation | Strong infrastructure isolation |
| Lower cost | Higher cost |
| Easier management | More operational complexity |
| Trusted tenants | Untrusted or regulated workloads |

---

# 5. Tenant Isolation Models

## Namespace Isolation

```
Cluster

├── Team A Namespace
├── Team B Namespace
└── Team C Namespace
```

---

## Node Isolation

```
Worker Node A

↓

Finance

Worker Node B

↓

AI

Worker Node C

↓

HR
```

Typically implemented using:

- Node labels
- Node affinity
- Taints and tolerations

---

## Cluster Isolation

```
Production

↓

Cluster A

Development

↓

Cluster B

Research

↓

Cluster C
```

---

# 6. Namespace-Based Isolation

Namespaces are the primary logical isolation mechanism in Kubernetes.

Example:

```bash
kubectl create namespace finance

kubectl create namespace hr

kubectl create namespace ai
```

---

## Benefits

- Resource organization
- RBAC scoping
- Quotas
- Policy application
- Cleaner operations

---

## Limitations

Namespaces do **not** provide complete security isolation on their own.

Combine them with:

- RBAC
- NetworkPolicies
- Pod Security Admission
- Resource quotas

---

# 7. ResourceQuota

A `ResourceQuota` limits resource consumption within a namespace.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota

metadata:
  name: finance-quota

spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi

    limits.cpu: "40"
    limits.memory: 80Gi

    pods: "100"
```

---

## Benefits

Prevents:

- Resource exhaustion
- Noisy neighbors
- Unlimited Pod creation

---

## Verify

```bash
kubectl get resourcequota

kubectl describe resourcequota
```

---

# 8. LimitRange

`LimitRange` sets default and maximum resource values for containers.

Example:

```yaml
apiVersion: v1
kind: LimitRange

metadata:
  name: default-limits
```

Typical settings include:

- Default CPU request
- Default memory request
- Maximum CPU
- Maximum memory

---

## Benefits

Ensures workloads have sensible resource requests and limits, improving scheduling and cluster stability.

---

# 9. Labels & Governance

Labels help organize resources across tenants.

Example:

```yaml
labels:

  team: finance

  environment: production

  cost-center: cc-101

  owner: platform
```

---

## Governance Benefits

- Cost allocation
- Reporting
- Automation
- Compliance
- Monitoring
- Chargeback and showback

---

## Recommended Labeling Strategy

| Label | Example |
|--------|---------|
| app | payment-api |
| team | finance |
| environment | production |
| owner | platform-team |
| cost-center | cc-101 |
| version | v2.3.1 |

Consistent labeling improves visibility and operational management.

---

# Multi-Tenant Architecture

```
                    Kubernetes Cluster
                           │
      ┌────────────────────┼────────────────────┐
      ▼                    ▼                    ▼
 Finance Namespace    HR Namespace       AI Namespace
      │                    │                    │
 ResourceQuota       ResourceQuota      ResourceQuota
 LimitRange          LimitRange         LimitRange
 RBAC                RBAC               RBAC
 NetworkPolicy       NetworkPolicy      NetworkPolicy
```

---

# 10. Part 1 Summary

You learned:

- Multi-tenancy fundamentals
- Soft vs hard multi-tenancy
- Tenant isolation models
- Namespace isolation
- ResourceQuota
- LimitRange
- Labels for governance

You now understand how Kubernetes enables multiple teams to share a cluster while applying foundational isolation and governance controls.


# K8S-31 – Multi-Tenancy & Governance (Part 2)

> **Class Date:** 23-Jul-2026
>
> **Module:** Kubernetes Platform Governance & Security

---

# 📚 Table of Contents

11. RBAC for Multi-Tenancy
12. Network Isolation
13. Pod Security Admission (PSA)
14. Admission Controllers
15. Policy as Code
16. OPA Gatekeeper
17. Kyverno
18. Namespace Governance
19. Production Governance Workflow
20. Part 2 Summary

---

# 11. RBAC for Multi-Tenancy

RBAC (Role-Based Access Control) ensures users only have the permissions they need.

## Principle of Least Privilege

Grant only the minimum permissions required.

Example:

```
Developer

↓

Read Pods
Create Deployments

✗ Cannot Delete Namespaces
✗ Cannot Modify Cluster Roles
```

---

## Typical Enterprise Roles

| Role | Permissions |
|------|-------------|
| Developer | Deploy applications, view logs |
| QA Engineer | Read workloads, execute tests |
| DevOps Engineer | Manage deployments, services, ingress |
| Platform Team | Cluster administration |
| Security Team | Audit policies, RBAC, security resources |

---

## Namespace-Scoped RBAC

```
Finance Namespace

↓

Role

↓

RoleBinding

↓

Finance Developers
```

A namespace `Role` cannot grant permissions outside its namespace.

---

## Cluster-Wide RBAC

```
Cluster

↓

ClusterRole

↓

ClusterRoleBinding

↓

Platform Administrators
```

Use cluster-wide permissions sparingly.

---

# 12. Network Isolation

Namespaces do **not** isolate network traffic by themselves.

Pods can generally communicate unless restricted.

---

## Without NetworkPolicy

```
Finance

↓

HR

↓

AI

↓

All Communication Allowed
```

---

## With NetworkPolicy

```
Finance

↓

Finance Only

HR

↓

HR Only

AI

↓

AI Only
```

Traffic is explicitly allowed where needed.

---

## Example Strategy

- Default deny ingress
- Allow only required namespaces
- Restrict external egress where appropriate
- Permit DNS traffic

---

## Production Recommendation

Every production namespace should have:

- Default deny ingress
- Default deny egress (where practical)
- Explicit allow rules

---

# 13. Pod Security Admission (PSA)

Pod Security Admission is the built-in Kubernetes mechanism for enforcing Pod security standards.

It replaces the deprecated PodSecurityPolicy (PSP).

---

## Profiles

| Profile | Purpose |
|----------|----------|
| Privileged | Minimal restrictions |
| Baseline | Prevent common privilege escalation |
| Restricted | Strong security controls |

---

## Example Namespace Labels

```yaml
pod-security.kubernetes.io/enforce: restricted
pod-security.kubernetes.io/audit: restricted
pod-security.kubernetes.io/warn: restricted
```

---

## Typical Restrictions

- No privileged containers
- Non-root execution
- Restricted Linux capabilities
- Safer volume usage
- Safer host namespace access

---

# 14. Admission Controllers

Admission controllers intercept API requests after authentication and authorization but before objects are persisted.

```
kubectl apply

↓

Authentication

↓

Authorization

↓

Admission Controllers

↓

etcd
```

---

## Common Built-in Admission Controllers

| Controller | Purpose |
|------------|----------|
| NamespaceLifecycle | Validates namespace lifecycle |
| LimitRanger | Applies default resource limits |
| ResourceQuota | Enforces quotas |
| MutatingAdmissionWebhook | Modifies objects |
| ValidatingAdmissionWebhook | Validates objects |

---

# 15. Policy as Code

Policy as Code allows governance rules to be defined declaratively and version-controlled.

Benefits:

- Repeatable
- Auditable
- Automated
- GitOps-friendly

---

## Examples

Require:

- Labels
- Resource requests
- Approved registries
- Non-root containers
- Approved namespaces

---

# 16. OPA Gatekeeper

OPA (Open Policy Agent) Gatekeeper enforces policies using the Kubernetes admission webhook mechanism.

Architecture:

```
kubectl apply

↓

Gatekeeper

↓

Constraint

↓

Allowed / Denied
```

---

## Example Policies

- Require labels
- Prevent privileged Pods
- Restrict hostPath volumes
- Require CPU and memory requests
- Restrict image registries

---

## Advantages

- Powerful policy language (Rego)
- Fine-grained controls
- Enterprise governance

---

# 17. Kyverno

Kyverno is a Kubernetes-native policy engine.

Unlike Gatekeeper, policies are written as Kubernetes-style YAML resources.

---

## Capabilities

- Validate resources
- Mutate resources
- Generate resources
- Verify container images (e.g., signatures)

---

## Example Use Cases

- Add missing labels automatically
- Block privileged Pods
- Require NetworkPolicies
- Enforce image registry rules

---

## Gatekeeper vs Kyverno

| Feature | Gatekeeper | Kyverno |
|----------|------------|----------|
| Policy Language | Rego | YAML |
| Mutate Resources | Limited compared to Kyverno | Yes |
| Generate Resources | No | Yes |
| Kubernetes-native Syntax | No | Yes |

---

# 18. Namespace Governance

Every namespace should have consistent governance.

Checklist:

✅ ResourceQuota

✅ LimitRange

✅ RBAC

✅ NetworkPolicy

✅ Pod Security Admission

✅ Monitoring

✅ Logging

✅ Backup Strategy (if applicable)

---

## Example

```
Finance Namespace

├── ResourceQuota
├── LimitRange
├── RBAC
├── NetworkPolicy
├── PSA Labels
└── Monitoring
```

---

# 19. Production Governance Workflow

```
Developer

↓

Git Commit

↓

Pull Request

↓

CI Validation

↓

Policy Checks

↓

GitOps Controller

↓

Admission Controllers

↓

Cluster

↓

Continuous Monitoring
```

Governance is enforced at multiple layers—not only inside the cluster.

---

# Governance Layers

```
Git
 │
 ▼
CI/CD Validation
 │
 ▼
GitOps
 │
 ▼
Admission Controllers
 │
 ▼
RBAC
 │
 ▼
NetworkPolicy
 │
 ▼
Pod Security Admission
 │
 ▼
Running Workloads
```

---

# Production Best Practices

## Namespace Standards

Every namespace should include:

- Quotas
- Limits
- RBAC
- NetworkPolicies
- PSA labels

---

## Secure Images

Allow images only from approved registries.

Examples:

- Internal registry
- Trusted public registries

---

## Least Privilege

Avoid granting:

- `cluster-admin`
- Wildcard (`*`) permissions

except where operationally required.

---

## Continuous Auditing

Regularly review:

- RBAC bindings
- Policies
- Privileged workloads
- Namespace configuration

---

## GitOps Integration

Store governance policies in Git.

Review policy changes through pull requests.

---

# 20. Part 2 Summary

You learned:

- RBAC strategies
- Network isolation
- Pod Security Admission
- Admission Controllers
- Policy as Code
- OPA Gatekeeper
- Kyverno
- Namespace governance
- Enterprise governance workflows

You now understand how production Kubernetes platforms automatically enforce security and governance across shared clusters.

# K8S-31 – Multi-Tenancy & Governance (Part 3)

> **Class Date:** 23-Jul-2026
>
> **Module:** Enterprise Kubernetes Governance

---

# 📚 Table of Contents

21. Cost Allocation & Chargeback
22. Resource Optimization
23. Multi-Cluster Governance
24. Platform Engineering
25. Enterprise Reference Architecture
26. Governance Automation
27. Hands-on Labs
28. Advanced Interview Questions
29. Governance Cheat Sheet
30. Chapter Summary

---

# 21. Cost Allocation & Chargeback

As Kubernetes adoption grows, organizations need visibility into **who is consuming resources** and **how much it costs**.

---

## Chargeback vs Showback

| Model | Description |
|--------|-------------|
| Showback | Report usage to teams without billing them |
| Chargeback | Bill teams based on actual resource consumption |

---

## Recommended Labels

```yaml
labels:
  team: finance
  cost-center: cc-101
  owner: finance-platform
  environment: production
  application: payment-api
```

These labels enable:

- Cost dashboards
- Financial reporting
- Budget tracking
- Resource ownership

---

## Typical Cost Metrics

Track:

- CPU usage
- Memory usage
- Storage consumption
- Network traffic
- Load balancer usage
- Persistent Volume usage

---

# 22. Resource Optimization

Efficient resource management improves cluster utilization and reduces cost.

---

## Common Problems

```
Requested CPU

██████████████

Actual CPU

██
```

Resources are requested but remain unused.

---

## Optimization Techniques

### Right-size workloads

Adjust CPU and memory requests based on observed usage.

---

### Autoscaling

Use:

- Horizontal Pod Autoscaler (HPA)
- Vertical Pod Autoscaler (VPA) where appropriate
- Cluster Autoscaler

---

### Remove Idle Resources

Delete:

- Unused namespaces
- Old deployments
- Orphaned PersistentVolumeClaims (after verifying they are no longer needed)
- Unused Services

---

## Continuous Monitoring

Use:

- Prometheus
- Grafana
- Cloud cost dashboards
- Resource usage reports

---

# 23. Multi-Cluster Governance

Large organizations rarely run a single Kubernetes cluster.

Example:

```
Production

↓

US-East Cluster

↓

Europe Cluster

↓

Asia Cluster
```

---

## Why Multiple Clusters?

- High availability
- Regional deployment
- Regulatory compliance
- Team isolation
- Scalability

---

## Governance Strategy

Apply consistent:

- RBAC
- Policies
- Labels
- Monitoring
- GitOps workflows
- Security controls

across every cluster.

---

# 24. Platform Engineering

Platform Engineering builds an internal platform that enables application teams to deploy safely and consistently.

```
Platform Team

↓

Internal Platform

↓

Application Teams
```

---

## Platform Responsibilities

- Kubernetes clusters
- CI/CD templates
- GitOps platform
- Security guardrails
- Monitoring
- Logging
- Networking
- Self-service capabilities

---

## Developer Experience

The goal is to reduce operational complexity for application teams while maintaining governance.

---

# 25. Enterprise Reference Architecture

```
                    Developers
                         │
                         ▼
                  Git Repository
                         │
                  Pull Request
                         │
                         ▼
                  CI/CD Validation
                         │
                         ▼
                  GitOps Controller
                         │
        ┌────────────────┼────────────────┐
        ▼                ▼                ▼
   Cluster A        Cluster B        Cluster C
        │                │                │
   RBAC             RBAC             RBAC
   Quotas           Quotas           Quotas
   PSA              PSA              PSA
   Policies         Policies         Policies
   Monitoring       Monitoring       Monitoring
```

---

# 26. Governance Automation

Governance should be automated wherever possible.

---

## Automated Checks

- Required labels
- Resource requests and limits
- Approved container registries
- Image signature verification (if adopted)
- Pod security policies via PSA and policy engines
- Namespace standards

---

## GitOps + Governance

```
Git Commit

↓

CI Validation

↓

Policy Validation

↓

GitOps

↓

Cluster

↓

Continuous Reconciliation
```

---

## Continuous Compliance

Regularly verify:

- RBAC permissions
- Namespace configuration
- Policy compliance
- Resource quotas
- Security posture

---

# Production Governance Checklist

## Namespace

- [ ] ResourceQuota
- [ ] LimitRange
- [ ] NetworkPolicy
- [ ] PSA labels
- [ ] Monitoring
- [ ] Logging

---

## Security

- [ ] RBAC
- [ ] Least privilege
- [ ] Approved registries
- [ ] Non-root containers
- [ ] Secrets managed securely

---

## Operations

- [ ] GitOps
- [ ] CI validation
- [ ] Backups
- [ ] Observability
- [ ] Incident response

---

# 27. Hands-on Labs

## Lab 1

Create three namespaces:

- finance
- hr
- ai

Apply:

- ResourceQuota
- LimitRange
- RBAC

---

## Lab 2

Create NetworkPolicies:

- Default deny
- Allow only required communication
- Validate connectivity

---

## Lab 3

Apply Pod Security Admission labels to a namespace.

Attempt to deploy a privileged Pod and observe the result.

---

## Lab 4

Install a policy engine (OPA Gatekeeper or Kyverno).

Implement policies to:

- Require labels
- Restrict privileged Pods
- Enforce resource requests

---

## Lab 5

Design a GitOps repository for a multi-team platform.

Separate:

- Applications
- Infrastructure
- Policies

---

# 28. Advanced Interview Questions

> **Note:** These topics are especially relevant for Platform Engineer, SRE, Cloud Architect, and Senior Kubernetes Administrator interviews.

## Beginner

1. What is multi-tenancy?
2. Compare soft and hard multi-tenancy.
3. Why use ResourceQuota?
4. What is a LimitRange?
5. What is showback?

---

## Intermediate

1. Explain namespace governance.
2. Compare OPA Gatekeeper and Kyverno.
3. How would you isolate teams in a shared cluster?
4. Explain chargeback and showback.
5. What is Policy as Code?

---

## Advanced

1. Design a Kubernetes platform for 500 development teams.
2. How would you govern multiple production clusters?
3. Explain enterprise Platform Engineering.
4. Design a secure governance pipeline.
5. How would you implement consistent policies across regions?

---

## Scenario-Based

### Scenario 1

One team consumes most of the cluster CPU.

How would you identify the issue and prevent recurrence?

---

### Scenario 2

A privileged Pod is deployed accidentally.

Which controls should have prevented this?

---

### Scenario 3

A namespace lacks NetworkPolicies and ResourceQuotas.

What risks does this introduce?

---

### Scenario 4

Your organization expands from one production cluster to five regions.

How would you maintain consistent governance?

---

### Scenario 5

Developers complain that governance policies slow them down.

How would you improve developer experience while preserving security?

---

# 29. Governance Cheat Sheet

## ResourceQuota

```bash
kubectl get resourcequota
kubectl describe resourcequota
```

---

## LimitRange

```bash
kubectl get limitrange
```

---

## RBAC

```bash
kubectl auth can-i create deployments \
  --namespace finance
```

---

## NetworkPolicy

```bash
kubectl get networkpolicy
```

---

## Pod Security Admission

```bash
kubectl get ns --show-labels
```

---

## Governance Principles

- Least privilege
- Policy as Code
- GitOps
- Continuous compliance
- Resource isolation
- Consistent labeling
- Automated enforcement

---

# 30. Chapter Summary

Congratulations! 🎉

You have completed **K8S-31 – Multi-Tenancy & Governance**.

In this chapter, you learned:

- Multi-tenancy fundamentals
- Soft vs hard isolation
- Namespace governance
- ResourceQuota
- LimitRange
- RBAC
- NetworkPolicies
- Pod Security Admission
- Admission Controllers
- OPA Gatekeeper
- Kyverno
- Policy as Code
- Cost allocation
- Chargeback and showback
- Platform Engineering
- Multi-cluster governance
- Governance automation
- Enterprise reference architecture
- Hands-on labs
- Interview questions

You now have a strong foundation for designing and operating **enterprise-grade Kubernetes platforms** that balance security, governance, scalability, and developer productivity.

