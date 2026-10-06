# K8S-12 - StatefulSets (Part 1)

> **Class Date:** 04-Jul-2026
>
> **Module:** StatefulSets


---

# 📚 Table of Contents

1. Introduction
2. Why Deployments Are Not Enough
3. What is a StatefulSet?
4. Stateful vs Stateless Applications
5. StatefulSet Architecture
6. Stable Pod Identity
7. Stable Network Identity
8. Headless Services
9. StatefulSet YAML Explained
10. Part 1 Summary

---

# 1. Introduction

Not all applications behave the same way.

Some applications can lose a Pod and continue running without issues.

Others depend on:

- Stable hostnames
- Persistent storage
- Predictable startup order

These are **stateful applications**.

Kubernetes provides **StatefulSets** to manage them.

---

# 2. Why Deployments Are Not Enough

Deployments work well for stateless applications.

Example:

```
Deployment

↓

frontend-6f4f8b7c5d-abc12

frontend-6f4f8b7c5d-xz901

frontend-6f4f8b7c5d-km458
```

If a Pod fails:

```
Old Pod Deleted

↓

New Pod Created

↓

New Random Name
```

This is acceptable for web applications.

---

## Problem for Databases

Imagine a MySQL cluster.

```
mysql-0

mysql-1

mysql-2
```

Each instance has a specific role.

If the Pod name changes randomly:

- Replication configuration breaks.
- Clients cannot rely on stable identities.
- Cluster membership becomes difficult.

Deployments cannot provide those guarantees.

---

# 3. What is a StatefulSet?

A **StatefulSet** is a Kubernetes workload API that manages applications requiring:

- Stable Pod names
- Stable network identities
- Persistent storage
- Ordered deployment
- Ordered termination

---

# Architecture

```
Client

↓

Headless Service

↓

StatefulSet

↓

mysql-0

mysql-1

mysql-2

↓

Persistent Volumes
```

---

# Characteristics

✔ Stable Pod identity

✔ Stable DNS

✔ Stable storage

✔ Ordered scaling

✔ Ordered updates

---

# 4. Stateful vs Stateless Applications

| Stateless | Stateful |
|-----------|----------|
| Deployment | StatefulSet |
| Random Pod names | Stable Pod names |
| Pods interchangeable | Pods have identity |
| Storage optional | Persistent storage required |
| Order not important | Order often important |

---

# Examples

## Stateless

- NGINX
- React
- Angular
- Spring Boot API
- Node.js API

---

## Stateful

- MySQL
- PostgreSQL
- MongoDB
- Kafka
- ZooKeeper
- Elasticsearch
- Cassandra
- Redis (persistent mode)

---

# 5. StatefulSet Architecture

```
                 Client

                    │

                    ▼

          Headless Service

                    │

     ┌──────────────┼──────────────┐

     ▼              ▼              ▼

  mysql-0        mysql-1        mysql-2

     │              │              │

     ▼              ▼              ▼

   PVC-0          PVC-1          PVC-2

     │              │              │

     ▼              ▼              ▼

   PV-0           PV-1           PV-2
```

Each Pod gets its own dedicated storage.

---

# 6. Stable Pod Identity

Unlike Deployments,

Pods in a StatefulSet have predictable names.

Example:

```
mysql-0

mysql-1

mysql-2
```

If `mysql-1` is deleted:

```
Deleted

↓

Recreated

↓

mysql-1
```

The name remains the same.

---

# Benefits

- Stable replication configuration
- Easier monitoring
- Predictable DNS
- Easier troubleshooting

---

# 7. Stable Network Identity

Each Pod receives a stable DNS name.

Pattern:

```
<pod-name>.<headless-service>.<namespace>.svc.cluster.local
```

Example:

```
mysql-0.mysql.default.svc.cluster.local

mysql-1.mysql.default.svc.cluster.local

mysql-2.mysql.default.svc.cluster.local
```

Applications can communicate using these predictable hostnames.

---

# DNS Flow

```
Application

↓

mysql-1.mysql.default.svc.cluster.local

↓

Headless Service

↓

mysql-1
```

---

# 8. Headless Services

A StatefulSet normally works together with a **Headless Service**.

Unlike a normal ClusterIP Service,

a Headless Service does **not** provide a virtual IP.

Instead, it returns the IP addresses of individual Pods.

---

## ClusterIP Service

```
Client

↓

ClusterIP

↓

Load Balancing

↓

Pods
```

---

## Headless Service

```
Client

↓

DNS Query

↓

Individual Pod IPs

↓

mysql-0

mysql-1

mysql-2
```

---

## Headless Service YAML

```yaml
apiVersion: v1
kind: Service

metadata:
  name: mysql

spec:
  clusterIP: None

  selector:
    app: mysql

  ports:
  - port: 3306
```

---

### Explanation

```yaml
clusterIP: None
```

This makes the Service **headless**, allowing DNS to return individual Pod addresses instead of a single virtual IP.

---

# 9. StatefulSet YAML Explained

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:
  name: mysql

spec:

  serviceName: mysql

  replicas: 3

  selector:
    matchLabels:
      app: mysql

  template:

    metadata:
      labels:
        app: mysql

    spec:

      containers:

      - name: mysql

        image: mysql:8.4
```

---

## serviceName

```yaml
serviceName: mysql
```

References the Headless Service.

---

## replicas

```yaml
replicas: 3
```

Creates:

```
mysql-0

mysql-1

mysql-2
```

---

## selector

Matches the Pod labels.

---

## template

Defines the Pod specification.

---

# Deployment vs StatefulSet

| Feature | Deployment | StatefulSet |
|----------|------------|-------------|
| Pod names | Random | Stable |
| DNS identity | No | Yes |
| Ordered rollout | No | Yes |
| Dedicated storage | No | Yes |
| Database workloads | ❌ | ✅ |

---

# 10. Part 1 Summary

In this section, you learned:

- Why Deployments are not suitable for many stateful workloads
- What StatefulSets are
- Stateful vs Stateless applications
- Stable Pod identities
- Stable DNS names
- Headless Services
- Basic StatefulSet YAML

You now understand the core concepts that make StatefulSets different from Deployments.


# K8S-12 - StatefulSets (Part 2)

---

# 📚 Table of Contents

11. Ordered Pod Creation
12. Ordered Pod Deletion
13. Ordered Rolling Updates
14. VolumeClaimTemplates
15. Persistent Storage Lifecycle
16. Pod Management Policy
17. Update Strategies
18. Production Examples
19. StatefulSet YAML Explained
20. Part 2 Summary

---

# 11. Ordered Pod Creation

Unlike Deployments,

StatefulSets create Pods **one at a time**.

Example:

```
Replicas = 3
```

Creation sequence:

```
mysql-0

↓

Running & Ready

↓

mysql-1

↓

Running & Ready

↓

mysql-2
```

Each Pod must become **Ready** before the next Pod is created.

---

## Why?

Many distributed systems require:

- Leader election
- Cluster formation
- Replication setup
- Bootstrap sequencing

Creating all Pods simultaneously could lead to initialization failures.

---

# 12. Ordered Pod Deletion

Deletion happens in the reverse order.

```
mysql-2

↓

mysql-1

↓

mysql-0
```

This helps protect cluster stability.

---

## Example

```
Scale Down

3 Pods

↓

2 Pods

↓

mysql-2 Removed

↓

mysql-0 and mysql-1 Continue Running
```

---

# 13. Ordered Rolling Updates

StatefulSets update Pods sequentially.

Example:

```
mysql-0

↓

Updated

↓

Ready

↓

mysql-1

↓

Updated

↓

Ready

↓

mysql-2
```

Only one Pod is updated at a time by default.

---

## Benefits

✔ Reduced risk

✔ Safer upgrades

✔ Easier rollback if problems are detected

---

# 14. VolumeClaimTemplates

Every StatefulSet Pod receives **its own PersistentVolumeClaim (PVC)**.

You define the template once,

and Kubernetes creates a unique PVC for each Pod.

---

## Architecture

```
StatefulSet

↓

VolumeClaimTemplate

↓

mysql-0

↓

mysql-data-mysql-0

↓

PV

----------------------

mysql-1

↓

mysql-data-mysql-1

↓

PV

----------------------

mysql-2

↓

mysql-data-mysql-2

↓

PV
```

---

## Example YAML

```yaml
volumeClaimTemplates:

- metadata:

    name: mysql-data

  spec:

    accessModes:

      - ReadWriteOnce

    storageClassName: standard

    resources:

      requests:

        storage: 20Gi
```

---

## What Kubernetes Creates

For:

```
mysql-0
```

PVC:

```
mysql-data-mysql-0
```

---

For:

```
mysql-1
```

PVC:

```
mysql-data-mysql-1
```

---

Each Pod gets its own dedicated storage.

---

# 15. Persistent Storage Lifecycle

Suppose:

```
mysql-1

↓

Pod Deleted
```

What happens?

```
New mysql-1

↓

Same PVC

↓

Same PV

↓

Database Preserved
```

The Pod is recreated,

but it reconnects to its original storage.

---

## Scaling Down

```
Replicas

3

↓

2
```

Pod:

```
mysql-2
```

is removed.

Its associated PVC is typically **retained** by default so that data is not lost unintentionally.

This allows:

- Future scale-up
- Manual recovery
- Data inspection

---

# 16. Pod Management Policy

StatefulSets support two Pod management policies.

---

## OrderedReady (Default)

```
mysql-0

↓

Ready

↓

mysql-1

↓

Ready

↓

mysql-2
```

Best for databases and clustered applications.

---

## Parallel

```
mysql-0

mysql-1

mysql-2
```

All Pods are created or deleted simultaneously.

Useful only when the application can safely handle parallel startup and shutdown.

---

## Example

```yaml
podManagementPolicy: Parallel
```

---

# 17. Update Strategies

## RollingUpdate (Default)

Pods are updated one by one.

```
mysql-0

↓

mysql-1

↓

mysql-2
```

This is the recommended strategy for most StatefulSets.

---

## OnDelete

No automatic updates occur.

```
Image Updated

↓

Nothing Happens

↓

Delete Pod Manually

↓

New Version Starts
```

Useful when administrators want complete control over update timing.

---

## Example

```yaml
updateStrategy:

  type: RollingUpdate
```

---

# 18. Production Examples

## MySQL Replication

```
mysql-0

Primary

↓

mysql-1

Replica

↓

mysql-2

Replica
```

Stable identities simplify replication configuration.

---

## Kafka

```
broker-0

broker-1

broker-2
```

Each broker keeps:

- Stable hostname
- Stable storage
- Stable broker ID

---

## MongoDB Replica Set

```
mongo-0

mongo-1

mongo-2
```

Replica set members depend on predictable identities.

---

## Elasticsearch

```
es-0

es-1

es-2
```

Each node stores shards on its own persistent volume.

---

# 19. StatefulSet YAML Explained

```yaml
apiVersion: apps/v1
kind: StatefulSet

metadata:

  name: mysql

spec:

  serviceName: mysql

  replicas: 3

  podManagementPolicy: OrderedReady

  updateStrategy:

    type: RollingUpdate

  selector:

    matchLabels:

      app: mysql

  template:

    metadata:

      labels:

        app: mysql

    spec:

      containers:

      - name: mysql

        image: mysql:8.4

        volumeMounts:

        - name: mysql-data

          mountPath: /var/lib/mysql

  volumeClaimTemplates:

  - metadata:

      name: mysql-data

    spec:

      accessModes:

      - ReadWriteOnce

      storageClassName: standard

      resources:

        requests:

          storage: 20Gi
```

---

## Key Fields

### serviceName

Links the StatefulSet to its Headless Service.

---

### podManagementPolicy

Controls creation and deletion order.

---

### updateStrategy

Defines how updates are performed.

---

### volumeClaimTemplates

Automatically creates one PVC per Pod.

---

### volumeMounts

Mounts the dedicated PVC into the container.

---

# Deployment vs StatefulSet (Advanced)

| Feature | Deployment | StatefulSet |
|---------|------------|-------------|
| Random Pod names | ✅ | ❌ |
| Stable Pod names | ❌ | ✅ |
| Dedicated PVC per Pod | ❌ | ✅ |
| Ordered startup | ❌ | ✅ |
| Ordered shutdown | ❌ | ✅ |
| Ordered updates | ❌ | ✅ |
| Best for databases | ❌ | ✅ |

---

# 20. Part 2 Summary

In this section, you learned:

- Ordered Pod creation
- Ordered Pod deletion
- Ordered rolling updates
- VolumeClaimTemplates
- Dedicated PVCs
- Persistent storage lifecycle
- Pod management policies
- Update strategies
- Production database examples
- Complete StatefulSet YAML

You now understand how StatefulSets provide stable identity, storage, and lifecycle management for stateful workloads.

# K8S-12 - StatefulSets (Part 3)

> **Class Date:** 04-Jul-2026

---

# 📚 Table of Contents

21. Scaling StatefulSets
22. Failure Recovery
23. Pod Rescheduling
24. Backup & Disaster Recovery
25. Production Best Practices
26. Common Mistakes
27. Troubleshooting
28. Hands-on Labs
29. CKA Exam Tips
30. Interview Questions
31. StatefulSet Cheat Sheet
32. Chapter Summary

---

# 21. Scaling StatefulSets

Unlike Deployments, StatefulSets scale in an ordered manner.

## Scale Up

Example:

```
Replicas = 3

↓

Replicas = 5
```

Creation order:

```
mysql-3

↓

Ready

↓

mysql-4

↓

Ready
```

The existing Pods are not recreated.

Each new Pod receives:

- A stable name
- A dedicated PVC
- A stable DNS entry

---

## Scale Down

Example:

```
Replicas = 5

↓

Replicas = 3
```

Removal order:

```
mysql-4

↓

mysql-3
```

Pods with lower ordinal numbers remain running.

---

## Important Note

Scaling down **does not normally delete the PVCs** created by `volumeClaimTemplates`.

Example:

```
mysql-4 Deleted

↓

PVC Retained

↓

PV Retained
```

If you scale up again,

```
mysql-4

↓

Existing PVC Reused

↓

Existing Data Available
```

---

# 22. Failure Recovery

## Pod Failure

Suppose:

```
mysql-1
```

crashes.

Kubernetes recreates:

```
mysql-1
```

—not—

```
mysql-new
```

This preserves:

- Pod identity
- DNS name
- Storage association

---

## Node Failure

Scenario:

```
Node-A

↓

mysql-0
```

Node-A becomes unavailable.

If the storage backend supports it and scheduling constraints allow, Kubernetes can schedule `mysql-0` on another node and reattach its persistent volume.

```
Node-B

↓

mysql-0

↓

Same PVC

↓

Same Data
```

The exact behavior depends on the CSI driver, storage backend, and access mode.

---

# 23. Pod Rescheduling

Flow:

```
Pod Fails

↓

Scheduler

↓

New Node Selected

↓

PVC Attached

↓

Container Starts

↓

Application Continues
```

Applications continue using the same persistent data.

---

# 24. Backup & Disaster Recovery

Persistent Volumes are **not backups**.

Use dedicated backup strategies.

---

## Common Options

- CSI Volume Snapshots
- Velero
- Cloud-native backup services
- Database-native backups (e.g., `mysqldump`, `pg_dump`, MongoDB tools)

---

## Backup Architecture

```
Database

↓

PVC

↓

Snapshot / Backup

↓

Object Storage / Backup Repository
```

---

## Disaster Recovery

Typical workflow:

```
Failure

↓

Restore Snapshot

↓

Create New Volume

↓

Attach to StatefulSet

↓

Application Restored
```

---

# 25. Production Best Practices

## Use Dedicated Storage

Each replica should have its own PVC.

Avoid sharing block storage between replicas unless the application and storage system explicitly support it.

---

## Monitor Storage

Track:

- Capacity
- IOPS
- Latency
- Filesystem usage
- Volume health

---

## Use Readiness Probes

Prevent Kubernetes from sending traffic before a database is ready.

Example:

```yaml
readinessProbe:
  tcpSocket:
    port: 3306
```

---

## Use Liveness Probes Carefully

Avoid aggressive liveness probes on databases.

Poorly configured probes can cause unnecessary restarts during recovery or heavy load.

---

## Plan Capacity

Monitor storage growth before disks become full.

---

## Test Recovery

A backup that has never been restored is **not** a verified backup.

Regularly test restore procedures.

---

# 26. Common Mistakes

## Using Deployment for Databases

```
Deployment

↓

Random Pod Identity

↓

Replication Problems
```

Use StatefulSets instead.

---

## Forgetting a Headless Service

Without a Headless Service:

- Stable DNS names are unavailable.
- Cluster communication may fail.

---

## Using emptyDir

```
Database

↓

emptyDir

↓

Pod Deleted

↓

Data Lost
```

Never use `emptyDir` for persistent databases.

---

## Deleting PVCs Accidentally

Deleting a PVC can permanently remove data depending on the reclaim policy and storage backend.

Always understand the storage lifecycle before deleting resources.

---

## Sharing One PVC

Most databases expect dedicated storage.

One PVC shared by multiple replicas is generally not appropriate unless the application and storage backend are specifically designed for that access pattern.

---

# 27. Troubleshooting

## Check StatefulSet

```bash
kubectl get statefulset
```

---

## Describe StatefulSet

```bash
kubectl describe statefulset mysql
```

---

## Check Pods

```bash
kubectl get pods
```

---

## Check PVCs

```bash
kubectl get pvc
```

---

## Check PVs

```bash
kubectl get pv
```

---

## Verify Headless Service

```bash
kubectl get svc
```

Ensure:

```
clusterIP: None
```

---

## Check DNS

```bash
kubectl exec -it mysql-0 -- nslookup mysql-1.mysql
```

---

## Check Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# Common Problems

| Problem | Cause | Solution |
|----------|-------|----------|
| Pod Pending | PVC not bound | Check StorageClass and PV |
| Pod CrashLoopBackOff | Application/configuration issue | Review logs and probes |
| DNS resolution fails | Missing or incorrect Headless Service | Verify Service configuration |
| PVC Pending | Storage unavailable | Check CSI driver and StorageClass |
| Volume attach failed | Node or storage issue | Check CSI controller logs and events |

---

# 28. Hands-on Labs

## Lab 1

Deploy a StatefulSet with:

- 3 replicas
- Headless Service
- `volumeClaimTemplates`

Observe:

```bash
kubectl get pods
kubectl get pvc
```

---

## Lab 2

Delete:

```
mysql-1
```

Verify:

- Same Pod name
- Same PVC
- Data remains

---

## Lab 3

Scale:

```
3

↓

5
```

Observe:

- `mysql-3`
- `mysql-4`
- New PVCs

---

## Lab 4

Scale:

```
5

↓

2
```

Verify:

- Higher ordinal Pods removed first
- PVCs remain

---

## Lab 5

Verify DNS

```bash
nslookup mysql-0.mysql
```

Repeat for the remaining replicas.

---

# 29. CKA Exam Tips

✔ Know the difference between:

- Deployment
- StatefulSet

✔ Memorize:

- Headless Service
- `clusterIP: None`
- `volumeClaimTemplates`

✔ Understand:

- Ordered startup
- Ordered shutdown
- Stable DNS
- Stable storage

✔ Practice:

```bash
kubectl get statefulset
kubectl describe statefulset
```

---

# 30. Interview Questions

## Beginner

1. What is a StatefulSet?
2. Why not use a Deployment for MySQL?
3. What is a Headless Service?
4. What is `volumeClaimTemplates`?
5. Why do StatefulSets use stable Pod names?

---

## Intermediate

1. Explain ordered Pod creation.
2. Explain ordered rolling updates.
3. How does StatefulSet scaling work?
4. Why are dedicated PVCs important?
5. What happens if a StatefulSet Pod is deleted?

---

## Advanced

1. Design a highly available PostgreSQL deployment on Kubernetes.
2. Explain StatefulSet recovery after node failure.
3. How do CSI drivers interact with StatefulSets?
4. How would you back up StatefulSet data?
5. Compare StatefulSets and Operators for database management.

---

## Scenario-Based

### Scenario 1

A MySQL replica loses its Pod.

How does Kubernetes recover it?

---

### Scenario 2

A PVC is stuck in `Pending`.

What would you investigate?

---

### Scenario 3

The Headless Service was accidentally deleted.

What impact would that have?

---

### Scenario 4

A team wants to use a Deployment for Kafka.

Would you recommend it? Why or why not?

---

### Scenario 5

A storage administrator asks why StatefulSets create one PVC per replica.

How would you explain the design?

---

# 31. StatefulSet Cheat Sheet

List StatefulSets:

```bash
kubectl get statefulset
```

Describe:

```bash
kubectl describe statefulset mysql
```

Scale:

```bash
kubectl scale statefulset mysql --replicas=5
```

List PVCs:

```bash
kubectl get pvc
```

List PVs:

```bash
kubectl get pv
```

Verify Headless Service:

```bash
kubectl get svc
```

Resolve DNS:

```bash
kubectl exec -it mysql-0 -- nslookup mysql-1.mysql
```

---

# 32. Key Takeaways

- StatefulSets provide stable Pod identities.
- Each Pod gets a dedicated PVC.
- Headless Services provide stable DNS.
- Pods are created, updated, and deleted in order by default.
- Scaling preserves storage identity.
- Persistent Volumes are not backups.
- Use CSI-backed storage for production.
- Monitor storage health and test recovery procedures.
- StatefulSets are the correct choice for most stateful distributed systems.

---

# 33. Chapter Summary

Congratulations! 🎉

You have completed **K8S-12 – StatefulSets**.

In this chapter, you learned:

- Why StatefulSets exist
- Stateful vs Stateless applications
- Stable Pod identities
- Stable DNS
- Headless Services
- `volumeClaimTemplates`
- Ordered startup and shutdown
- Ordered rolling updates
- Scaling behavior
- Persistent storage lifecycle
- Failure recovery
- Backup strategies
- Production best practices
- Troubleshooting
- Hands-on labs
- CKA preparation
- Interview questions

You now understand how Kubernetes manages databases and distributed applications reliably while preserving identity, networking, and persistent storage.