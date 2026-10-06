# K8S-23 - Security Contexts & Pod Security (Part 1)

> **Class Date:** 15-Jul-2026
>
> **Module:** Kubernetes Security
>
> **File Name:** `K8S-23_Security_Contexts_Pod_Security_15-Jul-2026.md`

---

# 📚 Table of Contents

1. Introduction
2. Why Security Contexts?
3. Pod Security Context vs Container Security Context
4. runAsUser
5. runAsGroup
6. fsGroup
7. First Security Context YAML
8. Part 1 Summary

---

# 1. Introduction

Containers run Linux processes.

Every process has:

- User ID (UID)
- Group ID (GID)
- File permissions
- Linux capabilities

Without restrictions, applications may run with more privileges than necessary.

Security Contexts reduce this risk.

---

# 2. Why Security Contexts?

Imagine a compromised web application.

If it runs as:

```
root (UID 0)
```

the attacker may have extensive control inside the container.

If it runs as:

```
UID 1000
```

the attack surface is significantly reduced.

---

## Principle of Least Privilege

Applications should run with:

- Minimum permissions
- Minimum capabilities
- No unnecessary privileges

---

# 3. Pod Security Context vs Container Security Context

Security settings can be defined at two levels.

| Level | Applies To |
|--------|------------|
| Pod Security Context | Default for all containers in the Pod |
| Container Security Context | Individual container only |

---

## Hierarchy

```
Pod

↓

Security Context

↓

Container A

Container B
```

A container can override certain Pod-level security settings with its own container-level configuration.

---

# 4. runAsUser

Controls which Linux user executes the container process.

Example:

```yaml
securityContext:

  runAsUser: 1000
```

Meaning:

```
Application

↓

Runs as

UID 1000
```

---

## Why Avoid Root?

Root inside a container is not equivalent to root on the host, but it still has elevated privileges within the container and may increase the impact of vulnerabilities or misconfigurations.

Running as a non-root user is a widely recommended security practice.

---

# 5. runAsGroup

Controls the primary Linux group.

Example:

```yaml
securityContext:

  runAsGroup: 3000
```

Result:

```
UID

1000

↓

Primary Group

3000
```

---

## Benefits

Useful for:

- Shared volumes
- File permissions
- Group ownership

---

# 6. fsGroup

Controls group ownership for supported mounted volumes.

Example:

```yaml
securityContext:

  fsGroup: 2000
```

Result:

```
Mounted Volume

↓

Group Ownership

2000
```

Applications running in the Pod can access shared storage more consistently when the storage plugin supports `fsGroup`.

---

## Typical Use Case

```
Shared Volume

↓

Multiple Containers

↓

Same Group Access
```

---

# 7. First Security Context YAML

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: secure-app

spec:

  securityContext:

    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000

  containers:

  - name: app

    image: nginx:1.27
```

---

# YAML Explained

## runAsUser

Runs processes as:

```
UID 1000
```

---

## runAsGroup

Primary group:

```
GID 3000
```

---

## fsGroup

Mounted volumes receive:

```
Group

2000
```

where supported.

---

# Verify

Apply:

```bash
kubectl apply -f secure-app.yaml
```

Verify:

```bash
kubectl exec -it secure-app -- id
```

Example output:

```text
uid=1000 gid=3000 groups=3000,2000
```

---

# Production Example

```
Web API

↓

runAsUser:1000

↓

Non-root

↓

Reduced Risk
```

---

# 8. Part 1 Summary

You learned:

- Security Context basics
- Pod vs Container Security Context
- `runAsUser`
- `runAsGroup`
- `fsGroup`
- First Security Context YAML

You now understand how Kubernetes controls the Linux identity of processes running inside containers.

# K8S-23 - Security Contexts & Pod Security (Part 2)

> **Class Date:** 15-Jul-2026

---

# 📚 Table of Contents

9. allowPrivilegeEscalation
10. Privileged Containers
11. readOnlyRootFilesystem
12. Linux Capabilities
13. Seccomp Profiles
14. AppArmor & SELinux
15. Production Security YAML
16. Part 2 Summary

---

# 9. allowPrivilegeEscalation

Linux allows processes to gain additional privileges under certain conditions.

Kubernetes lets you explicitly prevent this.

Example:

```yaml
securityContext:

  allowPrivilegeEscalation: false
```

---

## Why?

```
Application

↓

Compromised

↓

Attempt Privilege Escalation

↓

Blocked
```

---

## Production Recommendation

Unless an application specifically requires it:

```yaml
allowPrivilegeEscalation: false
```

---

# 10. Privileged Containers

A privileged container receives significantly expanded access to the host.

Example:

```yaml
securityContext:

  privileged: true
```

---

## Effect

A privileged container can access many host resources that are normally isolated.

```
Normal Container

↓

Limited Access

----------------

Privileged Container

↓

Much Greater Host Access
```

---

## Production Guidance

Avoid:

```yaml
privileged: true
```

unless there is a well-understood operational requirement.

Examples that may legitimately require elevated privileges include:

- Some storage plugins
- Some networking components
- Certain node management agents

---

# 11. readOnlyRootFilesystem

Applications often don't need to modify their root filesystem.

Enable:

```yaml
securityContext:

  readOnlyRootFilesystem: true
```

---

## Result

```
Container

↓

Cannot Modify

/

Filesystem
```

Applications should write only to:

- Writable mounted volumes
- `emptyDir`
- PersistentVolumes

---

## Benefits

- Reduces persistence opportunities for attackers
- Prevents accidental file modifications
- Encourages immutable container design

---

# 12. Linux Capabilities

Linux divides root privileges into smaller capabilities.

Instead of granting full root privileges, grant only what is required.

---

## Drop All Capabilities

```yaml
securityContext:

  capabilities:

    drop:

    - ALL
```

---

## Add One Capability

Example:

```yaml
securityContext:

  capabilities:

    drop:

    - ALL

    add:

    - NET_BIND_SERVICE
```

---

## Meaning

The application can bind to privileged ports (such as port 80) without receiving every Linux capability.

---

## Common Capabilities

| Capability | Purpose |
|------------|---------|
| NET_BIND_SERVICE | Bind to ports below 1024 |
| SYS_TIME | Change system time |
| SYS_ADMIN | Broad administrative capability (avoid unless absolutely necessary) |
| NET_ADMIN | Manage networking configuration |

---

## Production Recommendation

Start with:

```yaml
drop:
- ALL
```

Then add only the specific capabilities required.

---

# 13. Seccomp Profiles

Seccomp filters Linux system calls.

It reduces the kernel attack surface available to the container.

---

## Runtime Default

```yaml
securityContext:

  seccompProfile:

    type: RuntimeDefault
```

---

## Flow

```
Container

↓

System Call

↓

Seccomp

↓

Allowed or Blocked
```

---

## Profile Types

| Type | Description |
|------|-------------|
| RuntimeDefault | Container runtime's default seccomp profile |
| Localhost | Custom profile stored on the node |
| Unconfined | No seccomp filtering (generally avoid) |

---

## Production Recommendation

Prefer:

```yaml
RuntimeDefault
```

for most workloads.

---

# 14. AppArmor & SELinux

These are Linux Security Modules.

---

## AppArmor

Restricts application behavior using profiles.

Example (node-dependent):

```
Application

↓

AppArmor Profile

↓

Restricted Access
```

AppArmor availability depends on the Linux distribution and node configuration.

---

## SELinux

Uses security labels to control access between processes and resources.

Common on:

- Red Hat Enterprise Linux
- Fedora
- CentOS Stream
- Other SELinux-enabled systems

---

## Kubernetes

SecurityContext can include SELinux options when the underlying nodes support SELinux.

---

# 15. Production Security YAML

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: hardened-app

spec:

  securityContext:

    runAsNonRoot: true

    runAsUser: 1000

    fsGroup: 2000

  containers:

  - name: app

    image: nginx:1.27

    securityContext:

      allowPrivilegeEscalation: false

      readOnlyRootFilesystem: true

      capabilities:

        drop:

        - ALL

      seccompProfile:

        type: RuntimeDefault
```

---

# YAML Explained

## runAsNonRoot

Ensures the container does not start as UID 0.

---

## allowPrivilegeEscalation

Prevents gaining additional privileges.

---

## readOnlyRootFilesystem

Makes the root filesystem immutable.

---

## drop: ALL

Removes all Linux capabilities.

---

## RuntimeDefault

Applies the runtime's default seccomp profile.

---

# Hardened Container Checklist

```
✓ Non-root user

✓ runAsNonRoot=true

✓ allowPrivilegeEscalation=false

✓ readOnlyRootFilesystem=true

✓ Drop ALL capabilities

✓ RuntimeDefault seccomp

✓ Writable data stored only on mounted volumes
```

---

# 16. Part 2 Summary

You learned:

- allowPrivilegeEscalation
- privileged containers
- readOnlyRootFilesystem
- Linux capabilities
- Seccomp
- AppArmor
- SELinux
- Hardened security configuration

You now understand how to significantly reduce container attack surface using Kubernetes security settings.

# K8S-23 - Security Contexts & Pod Security (Part 3)

> **Class Date:** 15-Jul-2026

---

# 📚 Table of Contents

17. Pod Security Admission (PSA)
18. Pod Security Standards
19. Namespace Labels for PSA
20. Troubleshooting Security Contexts
21. Common Security Mistakes
22. Production Hardening Checklist
23. Hands-on Labs
24. CKA / CKS Tips
25. Interview Questions
26. SecurityContext Cheat Sheet
27. Real Production Case Studies
28. Chapter Summary

---

# 17. Pod Security Admission (PSA)

---

## What is Pod Security Admission?

Pod Security Admission (PSA) is a **built-in admission controller** that enforces security requirements on Pods **at creation or update time**.

```
kubectl apply

        │

        ▼

API Server

        │

        ▼

Pod Security Admission

        │

 Allow / Reject / Warn

        ▼

Pod Created
```

---

## Why PSA?

Without enforcement:

```
Developer

↓

Creates

↓

Privileged Container

↓

Runs Successfully
```

With PSA:

```
Developer

↓

Creates Privileged Pod

↓

Rejected
```

---

## Important

PodSecurityPolicy (PSP) is **deprecated and removed**.

Modern Kubernetes uses:

```
Pod Security Admission (PSA)
```

---

# 18. Pod Security Standards

There are three built-in policy levels.

| Level | Purpose |
|--------|---------|
| Privileged | Minimal restrictions |
| Baseline | Prevents known privilege escalations |
| Restricted | Strong security controls |

---

## Privileged

```
Everything

↓

Generally Allowed
```

Suitable only for trusted infrastructure components.

Examples:

- Some CNI plugins
- Some CSI drivers
- Node-level agents

---

## Baseline

Blocks many dangerous settings while remaining compatible with many applications.

Examples:

- Prevents several risky host configurations
- Limits dangerous capabilities

---

## Restricted

The recommended profile for most application workloads.

Typically expects:

- Non-root execution
- No privileged containers
- Restricted capabilities
- Seccomp profile
- Other hardened defaults

---

# 19. Namespace Labels for PSA

PSA is enabled by labeling namespaces.

---

## Enforce Mode

```bash
kubectl label namespace production \
pod-security.kubernetes.io/enforce=restricted
```

Pods violating the Restricted profile are rejected.

---

## Warn Mode

```bash
kubectl label namespace production \
pod-security.kubernetes.io/warn=restricted
```

Pods are created, but warnings are displayed.

Useful during migrations.

---

## Audit Mode

```bash
kubectl label namespace production \
pod-security.kubernetes.io/audit=restricted
```

Violations are recorded for auditing without blocking workloads.

---

## Version Pinning

You can pin policy behavior to a Kubernetes version.

Example:

```bash
kubectl label namespace production \
pod-security.kubernetes.io/enforce-version=latest
```

> Many organizations pin to a specific Kubernetes minor version instead of `latest` to keep enforcement behavior stable during upgrades.

---

# PSA Flow

```
Namespace

↓

Restricted

↓

New Pod

↓

Validate

↓

Allowed

or

Rejected
```

---

# 20. Troubleshooting Security Contexts

---

## Pod Does Not Start

Describe the Pod:

```bash
kubectl describe pod app
```

Look for:

- Admission errors
- Security validation failures
- Container runtime messages

---

## Verify Runtime Identity

```bash
kubectl exec app -- id
```

Example:

```
uid=1000

gid=3000
```

---

## Check Security Context

```bash
kubectl get pod app -o yaml
```

Verify:

- runAsUser
- runAsNonRoot
- fsGroup
- allowPrivilegeEscalation
- capabilities
- seccompProfile

---

## Common PSA Error

```
violates PodSecurity "restricted"
```

Possible causes:

- Running as root
- Privileged container
- Missing seccomp profile
- Disallowed capability

---

# 21. Common Security Mistakes

---

## Mistake 1

Running everything as:

```
root
```

---

## Mistake 2

Using:

```yaml
privileged: true
```

for ordinary applications.

---

## Mistake 3

Keeping:

```yaml
allowPrivilegeEscalation: true
```

without a valid requirement.

---

## Mistake 4

Granting unnecessary Linux capabilities.

Prefer:

```yaml
drop:
- ALL
```

---

## Mistake 5

Ignoring PSA warnings during deployment.

Warnings often indicate future enforcement failures.

---

# 22. Production Hardening Checklist

```
✓ runAsNonRoot=true

✓ runAsUser configured

✓ allowPrivilegeEscalation=false

✓ readOnlyRootFilesystem=true

✓ Drop ALL capabilities

✓ RuntimeDefault seccomp

✓ Dedicated ServiceAccount

✓ NetworkPolicy applied

✓ Restricted PSA

✓ Secrets stored securely

✓ Resource limits configured

✓ Liveness & Readiness probes configured
```

---

# 23. Hands-on Labs

---

## Lab 1

Deploy:

```yaml
runAsNonRoot: true
```

Verify:

```bash
kubectl exec app -- id
```

---

## Lab 2

Attempt to deploy:

```yaml
privileged: true
```

inside a namespace enforcing the Restricted profile.

Observe the rejection.

---

## Lab 3

Enable:

```
readOnlyRootFilesystem
```

Attempt to write to:

```
/tmp
```

If your application requires writable temporary storage, mount an `emptyDir` volume (or another appropriate writable volume) for that path.

---

## Lab 4

Drop all Linux capabilities.

Verify the application still functions.

---

## Lab 5

Label a namespace with:

```
Restricted PSA
```

Deploy:

- Secure Pod ✅
- Insecure Pod ❌

Observe the different outcomes.

---

# 24. CKA / CKS Tips

✔ Memorize:

- runAsUser
- runAsNonRoot
- fsGroup
- allowPrivilegeEscalation
- readOnlyRootFilesystem
- capabilities
- seccompProfile

✔ Understand:

- Privileged
- Baseline
- Restricted

✔ Practice:

```bash
kubectl describe pod
kubectl exec -- id
kubectl get pod -o yaml
kubectl label namespace
```

✔ Remember:

PSA operates at the **namespace level**, while SecurityContext is defined on the **Pod or container**.

---

# 25. Interview Questions

## Beginner

1. What is a SecurityContext?
2. Why should containers avoid running as root?
3. What is `runAsUser`?
4. What is `fsGroup`?
5. What is `allowPrivilegeEscalation`?

---

## Intermediate

1. Compare Pod Security Context and Container Security Context.
2. Explain Linux capabilities.
3. Why use `readOnlyRootFilesystem`?
4. What is a seccomp profile?
5. Explain Pod Security Admission.

---

## Advanced

1. Design a hardened Pod specification for a production API.
2. Explain the Restricted Pod Security Standard.
3. Compare PSA and the old PodSecurityPolicy.
4. How would you migrate namespaces from `warn` to `enforce` mode?
5. How would you balance security and application compatibility?

---

## Scenario-Based

### Scenario 1

A Pod fails with:

```
runAsNonRoot
```

What would you check first?

---

### Scenario 2

An application writes to the root filesystem and now fails after enabling:

```
readOnlyRootFilesystem
```

How would you redesign the workload?

---

### Scenario 3

A team wants to deploy privileged containers in production.

What questions would you ask before approving?

---

### Scenario 4

A namespace reports:

```
violates PodSecurity "restricted"
```

How would you identify and resolve the issue?

---

### Scenario 5

Design a secure deployment for an Internet-facing API.

Which SecurityContext settings would you enable?

---

# 26. SecurityContext Cheat Sheet

Run as non-root:

```yaml
runAsNonRoot: true
```

Specify UID:

```yaml
runAsUser: 1000
```

Specify GID:

```yaml
runAsGroup: 3000
```

Volume group:

```yaml
fsGroup: 2000
```

Disable privilege escalation:

```yaml
allowPrivilegeEscalation: false
```

Read-only root filesystem:

```yaml
readOnlyRootFilesystem: true
```

Drop capabilities:

```yaml
capabilities:
  drop:
    - ALL
```

Seccomp:

```yaml
seccompProfile:
  type: RuntimeDefault
```

Namespace enforcement:

```bash
kubectl label namespace production \
pod-security.kubernetes.io/enforce=restricted
```

---

# 27. Real Production Case Studies

## Banking Application

```
Internet

↓

Ingress

↓

API

↓

Database
```

Security:

- Restricted PSA
- Non-root containers
- Read-only root filesystem
- NetworkPolicies
- Dedicated ServiceAccounts

---

## Multi-Tenant Platform

Each tenant namespace:

```
Restricted PSA

↓

Dedicated RBAC

↓

Dedicated NetworkPolicies
```

Strong isolation between teams.

---

## CI/CD Platform

Build Pods:

- Short-lived
- Non-root
- Restricted capabilities
- No privilege escalation
- RuntimeDefault seccomp

Infrastructure components that require elevated privileges are isolated into dedicated namespaces with carefully reviewed access.

---

# 28. Chapter Summary

Congratulations! 🎉

You have completed **K8S-23 – Security Contexts & Pod Security**.

In this chapter, you learned:

- SecurityContext fundamentals
- Pod vs Container Security Context
- `runAsUser`
- `runAsGroup`
- `fsGroup`
- `allowPrivilegeEscalation`
- `privileged`
- `readOnlyRootFilesystem`
- Linux capabilities
- Seccomp
- AppArmor
- SELinux
- Pod Security Admission (PSA)
- Privileged, Baseline, and Restricted standards
- Troubleshooting
- Production hardening
- Hands-on labs
- CKA / CKS preparation
- Interview questions

You now have a strong foundation for securing Kubernetes workloads using modern Kubernetes security practices.

