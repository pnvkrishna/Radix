# K8S-27 – Kubernetes Troubleshooting (Production Debugging) (Part 2)

> **Class Date:** 19-Jul-2026
>
> **Module:** Kubernetes Troubleshooting

---

# 📚 Table of Contents

12. Pending Pods
13. FailedScheduling
14. Node Health Problems
15. Resource Pressure
16. Taints & Tolerations
17. Node Affinity Issues
18. DNS Troubleshooting
19. Service Connectivity
20. NetworkPolicy Issues
21. kubectl debug & Ephemeral Containers
22. Part 2 Summary

---

# 12. Pending Pods

A Pod in the **Pending** state has been accepted by the API server but has **not yet started running**.

```
kubectl apply

↓

API Server

↓

Scheduler

↓

Pending

↓

Running
```

If it never reaches **Running**, investigate why scheduling failed.

---

## First Commands

```bash
kubectl get pods

kubectl describe pod <pod-name>
```

Always check the **Events** section first.

---

## Common Reasons

- Insufficient CPU
- Insufficient Memory
- No matching node
- PVC not bound
- Taints
- Node affinity mismatch
- Image still pulling
- CNI initialization delay

---

# 13. FailedScheduling

Example Event:

```
Warning

FailedScheduling

0/3 nodes are available
```

This message tells you **why** the scheduler couldn't place the Pod.

---

## Example

```
0/3 nodes available

2 Insufficient memory

1 node had taint
```

The scheduler explains every rejection reason.

---

## Check Node Capacity

```bash
kubectl describe node <node-name>
```

Look for:

```
Capacity

Allocatable

Allocated resources
```

---

## Check Requests

```yaml
resources:

  requests:

    cpu: "2"

    memory: "8Gi"
```

If no node has enough allocatable resources, the Pod remains Pending.

---

## Possible Fixes

- Reduce requests
- Add worker nodes
- Enable Cluster Autoscaler
- Free resources
- Adjust scheduling constraints

---

# 14. Node Health Problems

Check nodes:

```bash
kubectl get nodes
```

Healthy:

```
Ready
```

Problem:

```
NotReady
```

---

## Describe Node

```bash
kubectl describe node <node-name>
```

Review:

- Conditions
- Events
- Allocated resources
- Taints

---

## Node Conditions

| Condition | Meaning |
|-----------|---------|
| Ready | Node is healthy |
| NotReady | Node unavailable |
| MemoryPressure | Low available memory |
| DiskPressure | Low available disk space |
| PIDPressure | Process ID exhaustion |
| NetworkUnavailable | Networking not configured |

---

# 15. Resource Pressure

```
Node

↓

Memory Full

↓

Eviction

↓

Pods Terminated
```

---

## MemoryPressure

Symptoms:

- Evicted Pods
- OOMKills
- Scheduling failures

---

## DiskPressure

Symptoms:

- Image pulls fail
- Containers fail to start
- Evictions

---

## PIDPressure

Symptoms:

- New processes cannot start
- Container startup failures

---

## Investigation

```bash
kubectl describe node
```

Look for:

```
Conditions

Events
```

---

# 16. Taints & Tolerations

A taint prevents Pods from being scheduled unless they tolerate it.

```
Node

↓

Taint

↓

Pod Rejected
```

---

## View Taints

```bash
kubectl describe node
```

Example:

```
node-role.kubernetes.io/control-plane:NoSchedule
```

---

## Add Toleration

```yaml
tolerations:

- key: node-role.kubernetes.io/control-plane

  operator: Exists

  effect: NoSchedule
```

---

# 17. Node Affinity Issues

Example:

```yaml
nodeAffinity:

  requiredDuringSchedulingIgnoredDuringExecution:
```

If no node satisfies the rule:

```
Pending
```

---

## Verify Labels

```bash
kubectl get nodes --show-labels
```

Compare node labels with the Pod's affinity rules.

---

# 18. DNS Troubleshooting

Pods usually resolve names through **CoreDNS**.

```
Application

↓

DNS Query

↓

CoreDNS

↓

Service IP
```

---

## Test DNS

```bash
kubectl exec -it <pod> -- nslookup kubernetes.default
```

or

```bash
kubectl exec -it <pod> -- getent hosts kubernetes.default
```

(`getent` may be available even when `nslookup` is not.)

---

## Check CoreDNS

```bash
kubectl get pods -n kube-system
```

Verify the CoreDNS Pods are Running.

---

## Verify Service

```bash
kubectl get svc -n kube-system
```

Expected:

```
kube-dns
```

---

## Common Problems

- CoreDNS crash
- NetworkPolicy blocking DNS
- Incorrect DNS configuration
- CNI networking issues

---

# 19. Service Connectivity

Suppose:

```
Pod A

↓

Service

↓

Pod B
```

Connection fails.

---

## Verify Service

```bash
kubectl get svc
```

---

## Verify Endpoints

```bash
kubectl get endpoints
```

or (modern API):

```bash
kubectl get endpointslices
```

EndpointSlices are the preferred scalable endpoint representation in modern Kubernetes.

---

## Verify Labels

Service selector:

```yaml
selector:

  app: backend
```

Pod label:

```yaml
labels:

  app: backend
```

Labels **must match**.

---

## Test Connectivity

```bash
kubectl exec -it <pod> -- curl http://service-name
```

If `curl` isn't available:

```bash
kubectl exec -it <pod> -- wget -qO- http://service-name
```

---

# 20. NetworkPolicy Issues

Symptoms:

- Pod cannot reach another Pod
- DNS blocked
- Database inaccessible

---

## Check Policies

```bash
kubectl get networkpolicy
```

---

## Describe

```bash
kubectl describe networkpolicy
```

---

## Verify

- Pod selectors
- Namespace selectors
- Ingress rules
- Egress rules

Remember:

NetworkPolicies are **allow rules**.

If a Pod is selected by a policy, traffic is denied unless explicitly allowed.

---

# 21. kubectl debug & Ephemeral Containers

Sometimes a running container is minimal and lacks troubleshooting tools.

Example:

```
Container

↓

No shell

↓

No ping

↓

No curl
```

Use:

```bash
kubectl debug <pod-name> -it --image=busybox
```

or

```bash
kubectl debug <pod-name> -it --image=nicolaka/netshoot
```

This launches an **ephemeral container** attached to the Pod for debugging.

> **Note:** Ephemeral containers are intended for debugging. They are not restarted, cannot define ports or probes, and are not part of the application's normal lifecycle.

---

## Useful Checks

Inside the debug container:

```bash
ip addr

ip route

nslookup kubernetes.default

wget -qO- http://service-name
```

---

# Troubleshooting Decision Tree

```
Pod Pending?
      │
      ▼
Describe Pod
      │
      ▼
FailedScheduling?
      │
      ├──────────────┐
      ▼              ▼
Resources       Taints/Affinity
      │              │
      ▼              ▼
Fix Requests   Fix Scheduling Rules
```

---

# 22. Part 2 Summary

You learned:

- Pending Pods
- FailedScheduling
- Node health
- MemoryPressure
- DiskPressure
- PIDPressure
- Taints & Tolerations
- Node Affinity
- DNS troubleshooting
- Service connectivity
- NetworkPolicy debugging
- `kubectl debug`
- Ephemeral containers

You now have a structured approach for diagnosing scheduling, node, DNS, and networking issues.

# K8S-27 – Kubernetes Troubleshooting (Production Debugging) (Part 3)

> **Class Date:** 19-Jul-2026
>
> **Module:** Kubernetes Troubleshooting

---

# 📚 Table of Contents

23. Persistent Volume (PV) & PVC Troubleshooting
24. Ingress Troubleshooting
25. TLS & Certificate Issues
26. Control Plane Troubleshooting
27. etcd Health Checks
28. kubelet Troubleshooting
29. CNI Troubleshooting
30. Production Incident Playbooks
31. Hands-on Labs
32. CKA / CKAD / CKS Interview Questions
33. Kubernetes Troubleshooting Cheat Sheet
34. Chapter Summary

---

# 23. Persistent Volume (PV) & PVC Troubleshooting

Storage issues often appear as Pods remaining in **Pending** or containers failing to start.

```
Application

↓

PersistentVolumeClaim

↓

PersistentVolume

↓

Storage Backend
```

---

## Check PVC Status

```bash
kubectl get pvc
```

Example:

```
NAME        STATUS

mysql-pvc   Pending
```

---

## Describe PVC

```bash
kubectl describe pvc mysql-pvc
```

Look for:

- StorageClass
- Requested capacity
- Events
- Binding failures

---

## Check PV

```bash
kubectl get pv
```

---

## Common Problems

| Problem | Cause | Fix |
|----------|-------|-----|
| PVC Pending | No matching PV or dynamic provisioner | Verify StorageClass and provisioner |
| AccessMode mismatch | PV and PVC differ | Align access modes |
| Capacity mismatch | Requested size too large | Resize or provision a larger volume |
| StorageClass mismatch | Different classes | Use the correct StorageClass |

---

## Verify StorageClass

```bash
kubectl get storageclass
```

---

# 24. Ingress Troubleshooting

```
Client

↓

Load Balancer

↓

Ingress Controller

↓

Service

↓

Pods
```

Traffic can fail at any layer.

---

## Verify Ingress

```bash
kubectl get ingress
```

---

## Describe Ingress

```bash
kubectl describe ingress
```

Check:

- Host
- Paths
- Backend service
- Events

---

## Verify Service

```bash
kubectl get svc
```

---

## Verify Endpoints

```bash
kubectl get endpointslices
```

Ensure backend Pods are healthy and selected.

---

## Verify Ingress Controller

```bash
kubectl get pods -n ingress-nginx
```

(Replace the namespace if using a different Ingress controller.)

---

# Common Causes

- Wrong host
- Incorrect path
- Missing backend Service
- No ready endpoints
- Controller not running
- DNS not pointing to the load balancer

---

# 25. TLS & Certificate Issues

Symptoms:

- Browser certificate warnings
- TLS handshake failures
- HTTPS unavailable

---

## Verify Secret

```bash
kubectl get secret
```

---

## Inspect Certificate

```bash
kubectl describe certificate
```

(If using Cert-Manager.)

---

## Verify Ingress TLS

```yaml
tls:
- hosts:
  - app.example.com

  secretName: app-tls
```

Ensure:

- Secret exists
- Hostnames match
- Certificate is valid

---

# 26. Control Plane Troubleshooting

The control plane manages the cluster.

```
API Server

↓

Scheduler

↓

Controller Manager

↓

etcd
```

---

## Check Control Plane Pods

```bash
kubectl get pods -n kube-system
```

In kubeadm-based clusters, these are often static Pods.

---

## Common Components

- kube-apiserver
- kube-scheduler
- kube-controller-manager
- etcd

---

## Inspect Logs

```bash
kubectl logs <control-plane-pod> -n kube-system
```

For kubeadm static Pods, logs may also be available via the node's container runtime or `journalctl`, depending on your environment.

---

# 27. etcd Health Checks

`etcd` stores the cluster's desired state.

```
kubectl

↓

API Server

↓

etcd
```

If etcd is unavailable, the control plane cannot persist or retrieve cluster state.

---

## Check etcd Pod

```bash
kubectl get pods -n kube-system
```

---

## Verify Health

Many environments use `etcdctl`.

Example:

```bash
etcdctl endpoint health
```

> Ensure the correct certificates and environment variables are configured when using `etcdctl` in secured production clusters.

---

## Check Disk Space

Low disk space can affect etcd performance.

Also monitor:

- Database size
- Compaction
- Defragmentation (as recommended by your Kubernetes distribution)

---

# 28. kubelet Troubleshooting

The kubelet runs on every node and manages Pods.

---

## Check Service

On systemd-based Linux systems:

```bash
systemctl status kubelet
```

---

## View Logs

```bash
journalctl -u kubelet
```

---

## Common Issues

- Node registration failures
- Certificate problems
- Container runtime connectivity
- Disk pressure
- Image pull failures

---

# 29. CNI Troubleshooting

The Container Network Interface (CNI) enables Pod networking.

```
Pod A

↓

CNI

↓

Pod B
```

---

## Verify CNI Pods

Example:

```bash
kubectl get pods -n kube-system
```

Look for your CNI components (for example, Cilium, Calico, Flannel, or another supported plugin).

---

## Symptoms

- Pods cannot communicate
- DNS failures
- Services unreachable
- Pods stuck in `ContainerCreating`

---

## Investigation

Check:

- CNI Pods
- Node readiness
- CNI logs
- NetworkPolicies
- Routes (if applicable)

---

# 30. Production Incident Playbooks

---

## Scenario 1 — Pod Pending

```
Describe Pod

↓

Events

↓

Scheduler Message

↓

Fix Resource or Scheduling Constraint

↓

Verify Running
```

---

## Scenario 2 — Application Unreachable

```
Ingress

↓

Service

↓

EndpointSlice

↓

Pod

↓

Container Logs
```

---

## Scenario 3 — CrashLoopBackOff

```
Logs

↓

Previous Logs

↓

Events

↓

Configuration

↓

Fix

↓

Redeploy
```

---

## Scenario 4 — DNS Failure

```
CoreDNS

↓

DNS Service

↓

Pod DNS Lookup

↓

NetworkPolicy

↓

CNI
```

---

## Scenario 5 — Node NotReady

```
Node Conditions

↓

kubelet

↓

Container Runtime

↓

Resources

↓

Recover Node
```

---

# Universal Debugging Checklist

```
1. kubectl get
        │
2. kubectl describe
        │
3. kubectl logs
        │
4. kubectl get events
        │
5. Verify Configuration
        │
6. Verify Networking
        │
7. Verify Storage
        │
8. Verify Node Health
        │
9. Verify Control Plane
        │
10. Confirm Resolution
```

---

# 31. Hands-on Labs

## Lab 1

Create a PVC with an incorrect StorageClass.

Observe why it remains Pending and fix it.

---

## Lab 2

Deploy an Ingress with an incorrect backend Service name.

Trace the request path and correct the configuration.

---

## Lab 3

Create an expired or invalid TLS Secret in a test environment.

Observe how the Ingress behaves and replace it with a valid certificate.

---

## Lab 4

Simulate a `CrashLoopBackOff`.

Use:

```bash
kubectl describe
kubectl logs --previous
```

to identify the root cause.

---

## Lab 5

Use `kubectl debug` with an ephemeral container to troubleshoot network connectivity inside a Pod.

---

# 32. CKA / CKAD / CKS Interview Questions

> **Note:** Troubleshooting is a major practical skill across CKA, CKAD, and CKS, even though the exact scenarios vary by certification.

## Beginner

1. What is `CrashLoopBackOff`?
2. What causes `ImagePullBackOff`?
3. Why does a Pod stay in `Pending`?
4. What does `kubectl describe` show?
5. Why are Events important?

---

## Intermediate

1. How do you troubleshoot Service connectivity?
2. Explain EndpointSlices.
3. How do you troubleshoot a PVC that remains Pending?
4. How do you debug DNS failures?
5. What is an ephemeral container?

---

## Advanced

1. Design a production troubleshooting workflow.
2. How would you debug intermittent network failures?
3. How would you investigate a `NotReady` node?
4. Explain etcd health verification.
5. How would you troubleshoot an unavailable control plane?

---

## Scenario-Based

### Scenario 1

Users report that HTTPS suddenly stopped working after an Ingress update.

Which components would you verify first?

---

### Scenario 2

A Deployment reports Ready, but clients receive connection failures.

How would you isolate the issue?

---

### Scenario 3

A Pod can reach some Services but not others.

Which networking layers would you inspect?

---

### Scenario 4

An application starts after several retries.

Which telemetry and Kubernetes resources would help explain why?

---

### Scenario 5

A worker node repeatedly transitions between `Ready` and `NotReady`.

What evidence would you collect before deciding whether to drain or replace the node?

---

# 33. Kubernetes Troubleshooting Cheat Sheet

Cluster:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get events --sort-by=.lastTimestamp
```

Describe:

```bash
kubectl describe pod <pod>
kubectl describe node <node>
kubectl describe pvc <pvc>
kubectl describe ingress <ingress>
```

Logs:

```bash
kubectl logs <pod>
kubectl logs --previous <pod>
kubectl logs -f <pod>
```

Debug:

```bash
kubectl exec -it <pod> -- /bin/sh
kubectl debug <pod> -it --image=nicolaka/netshoot
```

Networking:

```bash
kubectl get svc
kubectl get endpointslices
kubectl get networkpolicy
```

Storage:

```bash
kubectl get pv
kubectl get pvc
kubectl get storageclass
```

---

# 34. Chapter Summary

Congratulations! 🎉

You have completed **K8S-27 – Kubernetes Troubleshooting (Production Debugging)**.

In this chapter, you learned:

- A structured troubleshooting methodology
- Pod lifecycle failures
- CrashLoopBackOff
- ImagePullBackOff
- Scheduling failures
- Node health
- DNS debugging
- Service connectivity
- NetworkPolicy troubleshooting
- Storage troubleshooting
- Ingress and TLS debugging
- Control plane troubleshooting
- etcd basics
- kubelet troubleshooting
- CNI troubleshooting
- Production incident playbooks
- Hands-on labs
- Interview questions

You now have a practical framework for diagnosing and resolving many of the issues encountered in production Kubernetes environments.

