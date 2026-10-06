# K8S-22 - Network Policies (Part 1)

> **Class Date:** 14-Jul-2026
>
> **Module:** Networking & Security
>
> **File Name:** `K8S-22_Network_Policies_14-Jul-2026.md`

---

# 📚 Table of Contents

1. Introduction
2. Kubernetes Networking Recap
3. What is a NetworkPolicy?
4. How NetworkPolicies Work
5. Pod Selectors
6. Policy Types
7. First NetworkPolicy
8. Part 1 Summary

---

# 1. Introduction

Imagine an application:

```
Internet

↓

Frontend

↓

Backend

↓

Database
```

Should:

```
Frontend

↓

Database
```

communicate directly?

Usually:

```
NO
```

Should:

```
Backend

↓

Database
```

communicate?

```
YES
```

NetworkPolicies enforce these communication rules.

---

# 2. Kubernetes Networking Recap

By default, Kubernetes networking follows these principles:

- Every Pod receives its own IP address.
- Pods can generally communicate with other Pods across nodes.
- Services provide stable virtual IPs and DNS names.
- There is **no automatic network isolation** between Pods.

---

## Default Behavior

```
Frontend Pod

↓

Backend Pod

↓

Database Pod

↓

Monitoring Pod

↓

Everything Can Reach Everything
```

Without NetworkPolicies, the network is typically **fully open**.

---

# 3. What is a NetworkPolicy?

A NetworkPolicy controls:

- Which Pods can receive traffic (**Ingress**)
- Which Pods can send traffic (**Egress**)

It works by selecting Pods and defining allowed traffic.

---

## Firewall Analogy

```
Firewall

↓

Allow

↓

Deny

----------------

NetworkPolicy

↓

Allow

↓

Everything Else Blocked (for selected Pods and policy types)
```

---

# 4. How NetworkPolicies Work

A NetworkPolicy:

1. Selects one or more Pods.
2. Defines allowed traffic.
3. Traffic not explicitly allowed is denied **for the selected Pods and the policy types covered**.

> **Important:** NetworkPolicies are **allow rules**. There is no explicit "deny" rule. Denial occurs because traffic is not included in any allow rule.

---

## Architecture

```
Client Pod

↓

NetworkPolicy

↓

Backend Pod
```

The NetworkPolicy determines whether the traffic is permitted.

---

# 5. Pod Selectors

Every NetworkPolicy starts by selecting Pods.

Example:

```yaml
podSelector:

  matchLabels:

    app: backend
```

Meaning:

```
Only Pods

↓

app=backend

↓

Protected by This Policy
```

Pods that are not selected by the policy are unaffected by that policy.

---

# 6. Policy Types

There are two policy types.

| Type | Controls |
|------|----------|
| Ingress | Incoming traffic to selected Pods |
| Egress | Outgoing traffic from selected Pods |

You can use:

```yaml
policyTypes:

- Ingress
```

or:

```yaml
policyTypes:

- Egress
```

or both:

```yaml
policyTypes:

- Ingress
- Egress
```

---

# 7. First NetworkPolicy

Allow traffic to backend Pods only from frontend Pods.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: backend-policy

spec:

  podSelector:

    matchLabels:

      app: backend

  policyTypes:

  - Ingress

  ingress:

  - from:

    - podSelector:

        matchLabels:

          app: frontend
```

---

# YAML Explained

## podSelector

Targets:

```
Backend Pods
```

---

## policyTypes

```
Ingress
```

This policy controls incoming traffic.

---

## ingress.from

Allow traffic only from Pods labeled:

```
app=frontend
```

Traffic from other Pods is denied to the selected backend Pods.

---

# Architecture

```
Frontend

↓

Allowed

↓

Backend

----------------

Database

↓

Blocked

↓

Backend
```

(Assuming the database Pods are not labeled `app=frontend`.)

---

# Verify

Apply:

```bash
kubectl apply -f networkpolicy.yaml
```

List policies:

```bash
kubectl get networkpolicy
```

Describe:

```bash
kubectl describe networkpolicy backend-policy
```

---

# Important Requirement

NetworkPolicies are enforced only if your Container Network Interface (CNI) plugin supports them.

Common CNIs with NetworkPolicy support include:

- Calico
- Cilium
- Antrea
- Weave Net (legacy environments)

Always verify your chosen CNI's capabilities.

---

# 8. Part 1 Summary

You learned:

- Kubernetes networking defaults
- Why NetworkPolicies are needed
- Pod selectors
- Ingress policies
- Policy types
- Your first NetworkPolicy

You now understand how NetworkPolicies begin isolating communication between Pods.

# K8S-22 - Network Policies (Part 2)

> **Class Date:** 14-Jul-2026

---

# 📚 Table of Contents

9. Egress Policies
10. Namespace Selectors
11. IP Blocks
12. Combining Selectors
13. Default Deny Policies
14. DNS Considerations
15. Production NetworkPolicy Examples
16. Part 2 Summary

---

# 9. Egress Policies

So far, we controlled **incoming traffic (Ingress).**

Now let's control **outgoing traffic (Egress).**

Example:

```
Backend Pod

↓

Database

Allowed

----------------

Backend Pod

↓

Internet

Blocked
```

---

## Egress Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: backend-egress

spec:

  podSelector:

    matchLabels:

      app: backend

  policyTypes:

  - Egress

  egress:

  - to:

    - podSelector:

        matchLabels:

          app: database
```

---

## Meaning

Backend Pods can only send traffic to:

```
Database Pods
```

Other outbound traffic is denied for the selected Pods.

---

# 10. Namespace Selectors

Sometimes you want to allow communication from an **entire namespace**.

Example:

```
Namespace

monitoring

↓

Prometheus

↓

Application
```

---

## Label the Namespace

```bash
kubectl label namespace monitoring \
purpose=monitoring
```

---

## YAML

```yaml
from:

- namespaceSelector:

    matchLabels:

      purpose: monitoring
```

---

## Result

```
Monitoring Namespace

↓

Allowed

↓

Application Pods
```

Any Pod in a namespace labeled:

```
purpose=monitoring
```

is allowed by this rule.

---

# 11. IP Blocks

You can allow or restrict traffic from IP ranges.

Example:

```yaml
from:

- ipBlock:

    cidr: 192.168.1.0/24
```

---

## Excluding Addresses

```yaml
ipBlock:

  cidr: 10.0.0.0/8

  except:

  - 10.1.0.0/16
```

Meaning:

```
Allow

10.0.0.0/8

Except

10.1.0.0/16
```

---

## Common Use Cases

- Corporate office network
- VPN ranges
- On-premises data centers
- Trusted external services

---

# 12. Combining Selectors

You can combine selectors to create more precise rules.

Example:

```yaml
from:

- namespaceSelector:

    matchLabels:

      team: payments

  podSelector:

    matchLabels:

      app: frontend
```

---

## Meaning

Allow traffic only from:

```
Namespace

team=payments

AND

Pods

app=frontend
```

Both conditions must match.

---

# 13. Default Deny Policies

One of the most important security patterns.

---

## Default Deny Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: default-deny-ingress

spec:

  podSelector: {}

  policyTypes:

  - Ingress
```

---

## Meaning

```
All Pods

↓

No Incoming Traffic

↓

Unless Another Policy Allows It
```

---

## Default Deny Egress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

metadata:
  name: default-deny-egress

spec:

  podSelector: {}

  policyTypes:

  - Egress
```

---

## Meaning

```
All Pods

↓

No Outgoing Traffic

↓

Unless Explicitly Allowed
```

---

## Zero Trust Model

```
Default

↓

Deny Everything

↓

Allow Only Required Traffic
```

This is the recommended production approach for sensitive environments.

---

# 14. DNS Considerations

A common mistake:

```
Default Deny Egress

↓

DNS Blocked

↓

Application Cannot Resolve Services
```

---

## Why?

Pods usually resolve names through the cluster DNS service (commonly CoreDNS).

If egress to DNS is blocked:

```
payment-api.default.svc.cluster.local

↓

Resolution Fails
```

---

## Example DNS Rule

```yaml
egress:

- to:

  - namespaceSelector:

      matchLabels:

        kubernetes.io/metadata.name: kube-system

  ports:

  - protocol: UDP

    port: 53

  - protocol: TCP

    port: 53
```

> Adjust this rule if your DNS service runs in a different namespace or uses different labels.

---

# 15. Production NetworkPolicy Examples

---

## Example 1

Frontend

↓

Backend

Allowed

Backend

↓

Database

Allowed

Everything Else

↓

Blocked

---

## Example 2

Monitoring

↓

Application

Allowed

Application

↓

Monitoring

Allowed

Other Namespaces

↓

Blocked

---

## Example 3

Application

↓

Internet

Blocked

Application

↓

Internal API

Allowed

---

## Example 4

Admin VPN

↓

Application

Allowed

Public Internet

↓

Blocked

---

# Architecture

```
Internet

↓

Ingress Controller

↓

Frontend

↓

Backend

↓

Database
```

Policies:

- Internet → Ingress Controller ✅
- Frontend → Backend ✅
- Backend → Database ✅
- Frontend → Database ❌
- Database → Internet ❌

---

# 16. Part 2 Summary

You learned:

- Egress policies
- Namespace selectors
- IP blocks
- Combined selectors
- Default deny policies
- DNS considerations
- Production NetworkPolicy patterns

You now understand how to build secure communication paths between workloads using a zero-trust approach.

# K8S-22 - Network Policies (Part 3)

> **Class Date:** 14-Jul-2026

---

# 📚 Table of Contents

17. Troubleshooting NetworkPolicies
18. Testing Network Connectivity
19. Common NetworkPolicy Mistakes
20. CNI Considerations
21. Production Best Practices
22. Hands-on Labs
23. CKA / CKS Tips
24. Interview Questions
25. NetworkPolicy Cheat Sheet
26. Real Production Case Studies
27. Chapter Summary

---

# 17. Troubleshooting NetworkPolicies

When an application suddenly cannot communicate with another service after applying a NetworkPolicy, follow a structured approach.

---

## Step 1: Verify the Policy

List policies:

```bash
kubectl get networkpolicy
```

Describe a policy:

```bash
kubectl describe networkpolicy backend-policy
```

Verify:

- Pod selector
- Ingress rules
- Egress rules
- Policy types

---

## Step 2: Verify Pod Labels

```bash
kubectl get pods --show-labels
```

Example:

```
backend-7d9fd

app=backend

tier=api
```

A typo in a label or selector is one of the most common causes of policy failures.

---

## Step 3: Verify Namespace Labels

```bash
kubectl get namespaces --show-labels
```

If using a `namespaceSelector`, confirm the expected labels exist.

---

## Step 4: Test Connectivity

Create a temporary debugging Pod:

```bash
kubectl run net-test \
  --image=busybox:1.36 \
  --restart=Never \
  -it --rm -- sh
```

From inside the Pod:

```bash
wget -qO- http://backend
```

or, if available:

```bash
nc -zv backend 8080
```

---

## Step 5: Check DNS

```bash
nslookup backend
```

or:

```bash
getent hosts backend
```

If DNS fails, review your egress rules to ensure access to the cluster DNS service.

---

# Common Connectivity Problems

| Problem | Possible Cause | Solution |
|----------|----------------|----------|
| Cannot reach backend | Ingress policy blocks traffic | Review ingress allow rules |
| DNS fails | DNS egress blocked | Allow traffic to the DNS service |
| Namespace rule ineffective | Namespace labels do not match | Verify namespace labels |
| Pods unexpectedly communicate | No policy selects the Pods | Confirm `podSelector` matches the intended Pods |
| External API unreachable | Egress policy blocks outbound traffic | Add an appropriate egress rule |

---

# 18. Testing Network Connectivity

Useful tools:

```bash
kubectl exec -it <pod-name> -- sh
```

Inside the Pod:

```bash
wget -qO- http://service-name
```

```bash
nslookup service-name
```

```bash
getent hosts service-name
```

If your image includes networking tools:

```bash
nc -zv service-name 80
```

or:

```bash
curl http://service-name
```

---

## Verify Services

```bash
kubectl get svc
```

---

## Verify Endpoints

```bash
kubectl get endpoints
```

or:

```bash
kubectl get endpointslices
```

---

# 19. Common NetworkPolicy Mistakes

---

## Mistake 1

Assuming NetworkPolicies work without CNI support.

Only CNIs that implement NetworkPolicy enforcement can apply these rules.

---

## Mistake 2

Forgetting DNS.

```
Default Deny

↓

DNS Blocked

↓

Application Appears Broken
```

---

## Mistake 3

Selecting the Wrong Pods

Example:

```yaml
podSelector:

  matchLabels:

    app: back-end
```

Actual label:

```
app=backend
```

The policy selects no Pods.

---

## Mistake 4

Blocking Required Egress

Examples:

- Database access
- Cloud storage
- Identity providers
- External APIs

Map required outbound dependencies before applying restrictive egress policies.

---

## Mistake 5

Creating Overly Complex Policies

Prefer:

- Small
- Focused
- Well-documented

policies rather than one very large policy.

---

# 20. CNI Considerations

NetworkPolicies depend on your CNI implementation.

Examples:

| CNI | NetworkPolicy Support |
|------|-----------------------|
| Calico | ✅ |
| Cilium | ✅ |
| Antrea | ✅ |
| AWS VPC CNI | Requires additional NetworkPolicy support depending on deployment |
| Flannel | Basic networking only (does not enforce NetworkPolicies by itself) |

Always verify the capabilities and configuration of your chosen CNI.

---

# 21. Production Best Practices

---

## Start with Default Deny

```text
Default

↓

Deny

↓

Explicit Allow
```

This is the foundation of a zero-trust network.

---

## Label Consistently

Example:

```
app=frontend

app=backend

app=database

environment=production

team=payments
```

Consistent labels make policies easier to maintain.

---

## Separate Environments

Use namespaces for:

- Development
- Testing
- Staging
- Production

Then apply namespace-specific NetworkPolicies.

---

## Keep Policies Modular

Instead of one large policy:

- Frontend policy
- Backend policy
- Database policy
- Monitoring policy

This improves readability and maintenance.

---

## Test Before Production

Validate:

- DNS
- Service communication
- External dependencies
- Monitoring
- Health checks

before rolling policies into production.

---

# 22. Hands-on Labs

---

## Lab 1

Create:

```yaml
default-deny-ingress
```

Verify that selected Pods no longer accept incoming connections.

---

## Lab 2

Allow:

```
Frontend

↓

Backend
```

Confirm:

- Frontend succeeds.
- Other Pods are denied.

---

## Lab 3

Apply a default-deny egress policy.

Observe DNS failures.

Add a DNS allow rule.

Verify name resolution is restored.

---

## Lab 4

Use a `namespaceSelector`.

Allow only the monitoring namespace to access application metrics.

---

## Lab 5

Restrict outbound traffic so that an application can:

- Reach the database
- Resolve DNS

but cannot access arbitrary external destinations.

---

# 23. CKA / CKS Tips

✔ Understand:

- Ingress
- Egress
- Pod selectors
- Namespace selectors
- `ipBlock`

✔ Practice:

```bash
kubectl get networkpolicy
kubectl describe networkpolicy
kubectl get pods --show-labels
kubectl get namespaces --show-labels
```

✔ Remember:

- NetworkPolicies are allow rules.
- Policies affect only the Pods they select.
- Default deny is implemented by selecting Pods and defining no matching allow traffic for the chosen policy type.

---

# 24. Interview Questions

## Beginner

1. What is a NetworkPolicy?
2. What is the difference between ingress and egress?
3. What is a podSelector?
4. What is a namespaceSelector?
5. Why are NetworkPolicies important?

---

## Intermediate

1. Explain the default networking behavior in Kubernetes.
2. How would you isolate frontend and database Pods?
3. Why might DNS stop working after applying a default-deny egress policy?
4. How does `ipBlock` work?
5. How would you troubleshoot blocked traffic?

---

## Advanced

1. Design a zero-trust network architecture for a Kubernetes cluster.
2. Compare Calico and Cilium from a NetworkPolicy perspective.
3. Explain how you would secure a multi-tenant Kubernetes cluster.
4. How would you validate NetworkPolicies before a production rollout?
5. Design NetworkPolicies for a three-tier application.

---

## Scenario-Based

### Scenario 1

A backend API becomes unreachable immediately after applying a NetworkPolicy.

What would you check first?

---

### Scenario 2

Monitoring agents cannot scrape application metrics.

How would you design the required NetworkPolicy?

---

### Scenario 3

An application should communicate only with:

- Database
- DNS

Everything else should be blocked.

How would you implement this?

---

### Scenario 4

Developers report that Pods in different namespaces can still communicate.

Under what conditions is this expected?

---

### Scenario 5

Your company wants to adopt a zero-trust networking model.

What sequence of NetworkPolicies would you deploy?

---

# 25. NetworkPolicy Cheat Sheet

List policies:

```bash
kubectl get networkpolicy
```

Describe policy:

```bash
kubectl describe networkpolicy <name>
```

Show Pod labels:

```bash
kubectl get pods --show-labels
```

Show namespace labels:

```bash
kubectl get namespaces --show-labels
```

Debug connectivity:

```bash
kubectl exec -it <pod-name> -- sh
```

Check DNS:

```bash
nslookup service-name
```

Verify endpoints:

```bash
kubectl get endpoints
kubectl get endpointslices
```

---

# 26. Real Production Case Studies

## Banking Platform

```
Internet

↓

Ingress Controller

↓

API

↓

Database
```

Policies:

- Internet → Ingress Controller ✅
- Ingress Controller → API ✅
- API → Database ✅
- API → Internet ❌
- Database → Internet ❌

---

## Multi-Tenant SaaS

```
Namespace A

↓

Isolated

Namespace B

↓

Isolated
```

Only shared infrastructure (logging, monitoring, ingress) is permitted to communicate across namespaces.

---

## Monitoring Stack

```
Prometheus

↓

Scrape Metrics

↓

Applications
```

NetworkPolicies allow only the monitoring namespace to reach metrics endpoints.

---

# 27. Chapter Summary

Congratulations! 🎉

You have completed **K8S-22 – Network Policies**.

In this chapter, you learned:

- Kubernetes networking defaults
- NetworkPolicy fundamentals
- Ingress and egress policies
- Pod selectors
- Namespace selectors
- `ipBlock`
- Default deny strategy
- DNS considerations
- CNI requirements
- Troubleshooting
- Production best practices
- Hands-on labs
- CKA / CKS preparation
- Interview questions

You now understand how to design and troubleshoot secure, zero-trust communication between Kubernetes workloads.

