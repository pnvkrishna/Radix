# K8S-10 - Volumes & Persistent Storage (Part 1)

> **Class Date:** 02-Jul-2026
>
> **Module:** Kubernetes Storage


---

# 📚 Table of Contents

1. Introduction
2. Why Containers Lose Data
3. Container Filesystem
4. What is a Volume?
5. Kubernetes Volume Lifecycle
6. Volume Types Overview
7. emptyDir
8. hostPath
9. Volume YAML Explained
10. Summary

---

# 1. Introduction

Applications can be divided into two categories.

## Stateless Applications

Examples:

- NGINX
- Frontend applications
- API Gateways
- Authentication proxies

If a Pod is deleted,

another Pod can immediately replace it.

No important data is lost.

---

## Stateful Applications

Examples:

- MySQL
- PostgreSQL
- MongoDB
- Kafka
- Elasticsearch
- Redis (persistent mode)

These applications store important data.

If data disappears,

the application becomes unusable.

---

# Stateless vs Stateful

```
Stateless

Pod Deleted

↓

New Pod

↓

Works Normally

----------------------------

Stateful

Pod Deleted

↓

Database Files Lost

↓

Application Failure
```

---

# 2. Why Containers Lose Data

Containers use a writable filesystem.

Example

```
Container

↓

Create File

↓

/data/users.db
```

Everything works.

But what happens if the container is deleted?

```
Container Deleted

↓

Filesystem Deleted

↓

users.db Lost
```

The data disappears.

---

# Kubernetes Example

```
Pod

↓

Container

↓

Filesystem

↓

Temporary
```

When the Pod is recreated,

a new container gets a new writable layer.

The previous writable layer is gone.

---

# Real World Example

Imagine writing important notes on a whiteboard.

```
Meeting

↓

Whiteboard

↓

Erase Board

↓

Notes Gone
```

Instead,

store them in a notebook.

```
Notebook

↓

Reusable

↓

Persistent
```

Volumes act like the notebook.

---

# 3. Container Filesystem

Every container has its own filesystem.

```
Container

├── /bin

├── /etc

├── /usr

├── /var

└── Writable Layer
```

The writable layer is **ephemeral**.

---

# Image Layers vs Writable Layer

```
Container Image

↓

Read Only Layers

↓

Writable Layer

↓

Application Writes Here
```

Only the writable layer changes at runtime.

If the container is removed,

the writable layer disappears.

---

# 4. What is a Volume?

A Volume provides storage that is attached to a Pod.

Instead of writing data only to the container's writable layer,

the application writes to the Volume.

```
Application

↓

Volume

↓

Storage
```

---

# Volume Architecture

```
Container

↓

Volume Mount

↓

Volume

↓

Storage
```

The container reads and writes through the mounted path.

---

# Why Volumes?

Without Volume

```
Container

↓

File

↓

Container Deleted

↓

File Lost
```

With Volume

```
Container

↓

Volume

↓

Storage

↓

Container Deleted

↓

Data Still Exists
```

**Note:** This persistence depends on the **type of volume**. For example, `emptyDir` exists only for the lifetime of a Pod, while Persistent Volumes can outlive Pods. We'll cover those differences shortly.

---

# 5. Kubernetes Volume Lifecycle

```
Pod Created

↓

Volume Mounted

↓

Application Uses Storage

↓

Pod Deleted

↓

Volume Behavior Depends on Volume Type
```

Different volume types have different lifecycles.

---

# 6. Volume Types

Common Kubernetes volume types include:

```
emptyDir

hostPath

configMap

secret

persistentVolumeClaim

projected

downwardAPI

CSI-backed volumes
```

We'll focus first on the foundational types.

---

# 7. emptyDir

One of the simplest volume types.

Created automatically when a Pod starts.

```
Pod Created

↓

emptyDir Created
```

All containers inside the Pod can share it.

---

# Architecture

```
Pod

├── Container A

├── Container B

└── emptyDir
```

---

# Characteristics

✔ Shared between containers in the same Pod.

✔ Created when Pod starts.

✔ Deleted when the Pod is permanently removed from the node.

✔ Good for temporary files.

---

# Common Use Cases

- Cache
- Temporary Downloads
- Intermediate Processing
- Scratch Space
- Shared Files Between Sidecar Containers

---

# emptyDir Example

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: shared-data

spec:

  volumes:

  - name: cache

    emptyDir: {}

  containers:

  - name: app

    image: nginx

    volumeMounts:

    - name: cache

      mountPath: /cache

  - name: helper

    image: busybox

    command: ["sleep","3600"]

    volumeMounts:

    - name: cache

      mountPath: /cache
```

Both containers share:

```
/cache
```

---

# emptyDir Lifecycle

```
Pod Starts

↓

emptyDir Created

↓

Containers Share Files

↓

Pod Deleted

↓

Data Removed
```

---

# Advantages

✔ Very Fast

✔ Simple

✔ Shared Storage

✔ No External Storage Required

---

# Limitations

❌ Data is temporary.

❌ Not suitable for databases.

---

# 8. hostPath

`hostPath` mounts a directory from the Kubernetes node into the Pod.

```
Node Filesystem

↓

hostPath

↓

Pod
```

---

# Example

Node

```
/var/log
```

Mounted into Pod

```
/logs
```

The application reads the node's log files.

---

# YAML Example

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: hostpath-demo

spec:

  volumes:

  - name: logs

    hostPath:

      path: /var/log

      type: Directory

  containers:

  - name: app

    image: nginx

    volumeMounts:

    - name: logs

      mountPath: /logs

      readOnly: true
```

---

# Common Use Cases

- Log Collection Agents
- Node Monitoring
- Debugging
- Accessing Hardware-Specific Files

Examples:

- Fluent Bit
- Fluentd
- Prometheus Node Exporter (depending on deployment)
- Some monitoring agents

---

# Advantages

✔ Direct node access.

✔ Useful for system-level workloads.

---

# Limitations

❌ Tightly coupled to a specific node.

❌ Reduces workload portability.

❌ Requires careful security review because it exposes part of the host filesystem.

Avoid `hostPath` for general application data in production unless there is a clear operational need.

---

# 9. Volume YAML Explained

Example

```yaml
volumes:

- name: storage

  emptyDir: {}
```

---

## name

```yaml
name: storage
```

Volume identifier inside the Pod.

---

## emptyDir

```yaml
emptyDir: {}
```

Create an empty temporary directory.

---

## Mount

```yaml
volumeMounts:

- name: storage

  mountPath: /data
```

This mounts the volume into:

```
/data
```

inside the container.

---

# 10. Part 1 Summary

In this section, you learned:

- Why containers lose data
- Container writable layers
- Stateless vs stateful applications
- What a Volume is
- Volume lifecycle
- Common volume types
- `emptyDir`
- `hostPath`
- Volume YAML basics

You now understand why Kubernetes needs Volumes and the difference between temporary Pod storage and more durable storage solutions.

# K8S-10 - Volumes & Persistent Storage (Part 2)

---

# 📚 Table of Contents

11. Persistent Volumes (PV)
12. Persistent Volume Claims (PVC)
13. PV-PVC Binding Process
14. Static vs Dynamic Provisioning
15. StorageClasses
16. Container Storage Interface (CSI)
17. Access Modes
18. Reclaim Policies
19. PV & PVC YAML Explained
20. Part 2 Summary

---

# 11. Persistent Volume (PV)

A **Persistent Volume (PV)** is a piece of storage managed by Kubernetes.

Unlike `emptyDir`, a PV is **independent of a Pod's lifecycle**.

```
Application

↓

Pod

↓

Persistent Volume Claim (PVC)

↓

Persistent Volume (PV)

↓

Storage Backend
```

A Pod does **not** access a PV directly. It requests storage through a PVC.

---

# Characteristics of a PV

✔ Cluster-level resource

✔ Independent of Pods

✔ Can outlive Pod deletion

✔ Backed by different storage systems

Examples:

- AWS EBS
- Azure Managed Disks
- Google Persistent Disk
- NFS
- Ceph
- SAN
- Local storage
- CSI drivers

---

# PV Architecture

```
Pod

↓

PVC

↓

Persistent Volume

↓

Disk / Cloud Storage
```

---

# 12. Persistent Volume Claim (PVC)

A **Persistent Volume Claim (PVC)** is a request for storage.

Think of it like requesting a virtual machine.

You don't ask for:

```
Disk Serial Number
```

Instead you ask for:

- Size
- Performance
- Access mode

Kubernetes finds suitable storage.

---

# Example

```
Application

↓

Needs

20Gi

↓

PVC

↓

Kubernetes

↓

Matching PV
```

---

# Why Use PVC?

Applications should not know where storage is physically located.

This abstraction provides portability across environments.

---

# 13. PV–PVC Binding Process

The binding process works like this:

```
Application

↓

PVC Created

↓

Kubernetes Searches

↓

Matching PV

↓

Bound

↓

Pod Starts
```

If no suitable PV exists:

```
PVC

↓

Pending
```

until storage becomes available.

---

# Binding Requirements

For a PV to bind to a PVC, Kubernetes checks compatibility, including:

- Requested storage capacity
- Access modes
- StorageClass (when applicable)

---

# 14. Static vs Dynamic Provisioning

## Static Provisioning

Administrator creates the PV first.

```
Admin

↓

Create PV

↓

User Creates PVC

↓

Bound
```

Example use cases:

- On-premises environments
- Pre-provisioned storage

---

## Dynamic Provisioning

Administrator creates a **StorageClass**.

```
User Creates PVC

↓

StorageClass

↓

CSI Driver

↓

New Disk

↓

PV Created Automatically

↓

PVC Bound
```

This is the preferred approach in most modern Kubernetes clusters.

---

# Static vs Dynamic

| Feature | Static | Dynamic |
|----------|---------|----------|
| Admin creates PV | ✅ | ❌ |
| Automatic storage creation | ❌ | ✅ |
| Cloud-friendly | Limited | ✅ |
| Recommended today | Rarely | ✅ |

---

# 15. StorageClass

A **StorageClass** defines **how storage should be provisioned**.

Examples:

- SSD
- HDD
- Premium SSD
- Standard Disk

---

# Architecture

```
PVC

↓

StorageClass

↓

CSI Driver

↓

Cloud Storage

↓

PV
```

---

## Example StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass

metadata:
  name: fast-storage

provisioner: ebs.csi.aws.com

parameters:
  type: gp3

allowVolumeExpansion: true

reclaimPolicy: Delete

volumeBindingMode: WaitForFirstConsumer
```

---

## Important Fields

### provisioner

Specifies the CSI driver.

Example:

```
ebs.csi.aws.com
```

---

### allowVolumeExpansion

```yaml
allowVolumeExpansion: true
```

Allows supported volumes to be expanded after creation.

---

### reclaimPolicy

Determines what happens after the PVC is released.

We'll discuss this in detail shortly.

---

### volumeBindingMode

```yaml
WaitForFirstConsumer
```

Delays volume provisioning until Kubernetes knows where the Pod will be scheduled.

This helps place storage in the correct availability zone or topology.

---

# 16. Container Storage Interface (CSI)

CSI is the standard interface between Kubernetes and storage providers.

Before CSI:

Every storage vendor required Kubernetes-specific integrations.

After CSI:

Every storage vendor implements the CSI specification.

```
Pod

↓

PVC

↓

StorageClass

↓

CSI Driver

↓

Storage System
```

---

# Popular CSI Drivers

- AWS EBS CSI Driver
- AWS EFS CSI Driver
- Azure Disk CSI Driver
- Azure File CSI Driver
- Google Persistent Disk CSI Driver
- Ceph CSI
- NFS CSI
- VMware vSphere CSI

---

# Why CSI?

✔ Vendor-independent

✔ Extensible

✔ Standardized

✔ Supports snapshots and expansion (depending on the driver)

---

# 17. Access Modes

Access modes describe **how a volume may be mounted**.

---

## ReadWriteOnce (RWO)

```
One Node

↓

Read + Write
```

Most common.

Supported by many block storage systems.

---

## ReadOnlyMany (ROX)

```
Many Nodes

↓

Read Only
```

Useful for shared reference data.

---

## ReadWriteMany (RWX)

```
Many Nodes

↓

Read + Write
```

Common with shared filesystems such as:

- NFS
- Azure Files
- Amazon EFS
- CephFS

---

## ReadWriteOncePod (RWOP)

```
One Pod

↓

Read + Write
```

Provides stronger exclusivity than RWO by ensuring only a single Pod can mount the volume for read/write at a time.

Useful for certain stateful workloads.

---

# Access Mode Comparison

| Mode | One Node | Many Nodes | Read | Write |
|------|----------|------------|------|-------|
| RWO | ✅ | ❌ | ✅ | ✅ |
| ROX | ✅ | ✅ | ✅ | ❌ |
| RWX | ✅ | ✅ | ✅ | ✅ |
| RWOP | Single Pod | ❌ | ✅ | ✅ |

> **Note:** The access modes supported depend on the underlying storage backend and CSI driver.

---

# 18. Reclaim Policies

When a PVC is deleted,

what should happen to the storage?

---

## Delete

```
PVC Deleted

↓

PV Deleted

↓

Storage Deleted
```

Common for dynamically provisioned cloud volumes.

---

## Retain

```
PVC Deleted

↓

PV Remains

↓

Administrator Reviews

↓

Manual Cleanup or Reuse
```

Useful when preserving data is important.

---

## Recycle (Historical)

Older Kubernetes versions supported `Recycle`.

This policy has been **removed** and should not be used in modern clusters.

---

# 19. PV and PVC YAML

## Persistent Volume

```yaml
apiVersion: v1
kind: PersistentVolume

metadata:
  name: pv-demo

spec:
  capacity:
    storage: 20Gi

  accessModes:
    - ReadWriteOnce

  storageClassName: standard

  persistentVolumeReclaimPolicy: Retain

  hostPath:
    path: /data/pv
```

---

## Explanation

### capacity

```yaml
storage: 20Gi
```

Maximum storage capacity.

---

### accessModes

```yaml
ReadWriteOnce
```

Specifies how the volume may be mounted.

---

### storageClassName

Matches compatible PVCs.

---

### persistentVolumeReclaimPolicy

Controls the fate of the storage after release.

---

### hostPath

Used here for demonstration.

In production, cloud or network-backed CSI storage is generally preferred.

---

# Persistent Volume Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: pvc-demo

spec:
  accessModes:
    - ReadWriteOnce

  storageClassName: standard

  resources:
    requests:
      storage: 20Gi
```

---

## Binding Result

```
PVC

↓

Bound

↓

PV

↓

Storage Ready

↓

Pod Can Mount
```

---

# 20. Part 2 Summary

In this section, you learned:

- Persistent Volumes (PV)
- Persistent Volume Claims (PVC)
- PV–PVC binding
- Static provisioning
- Dynamic provisioning
- StorageClasses
- CSI
- Access modes
- Reclaim policies
- PV and PVC YAML

You now understand how Kubernetes abstracts storage and dynamically provisions persistent volumes for applications.

# K8S-10 - Volumes & Persistent Storage (Part 3)

---

# 📚 Table of Contents

21. Volume Expansion
22. Volume Snapshots
23. Ephemeral CSI Volumes
24. Stateful Application Storage
25. Storage Performance Best Practices
26. Storage Troubleshooting
27. Hands-on Labs
28. CKA Exam Tips
29. Interview Questions
30. Storage Cheat Sheet
31. Chapter Summary

---

# 21. Volume Expansion

Applications grow over time.

Example:

```
Database

↓

20Gi

↓

100Gi

↓

500Gi
```

Instead of creating a new volume,

many CSI drivers support expanding an existing volume.

---

## Requirements

- StorageClass supports expansion.
- CSI driver supports expansion.
- Underlying storage platform supports expansion.

Example StorageClass:

```yaml
allowVolumeExpansion: true
```

---

## Expand PVC

Original

```yaml
resources:
  requests:
    storage: 20Gi
```

Updated

```yaml
resources:
  requests:
    storage: 50Gi
```

Apply:

```bash
kubectl apply -f pvc.yaml
```

Kubernetes requests expansion from the CSI driver.

> **Note:** Some filesystems expand online, while others may require filesystem resizing or a Pod restart depending on the storage driver and filesystem.

---

# 22. Volume Snapshots

Snapshots capture the state of a volume at a point in time.

```
Volume

↓

Snapshot

↓

Restore

↓

New Volume
```

Common uses:

- Backup
- Disaster Recovery
- Testing
- Cloning environments

---

## Snapshot Architecture

```
PVC

↓

CSI Driver

↓

VolumeSnapshot

↓

Storage Provider
```

Snapshots require:

- CSI snapshot support
- Snapshot controller
- Snapshot CRDs installed in the cluster

---

## Benefits

✔ Fast backups

✔ Easy restore

✔ Safe testing

✔ Disaster recovery

---

# 23. Ephemeral CSI Volumes

Sometimes storage is needed only while a Pod exists.

```
Pod Created

↓

CSI Ephemeral Volume

↓

Application Uses Storage

↓

Pod Deleted

↓

Storage Removed
```

Typical use cases:

- Temporary caches
- Scratch space
- Short-lived processing

These differ from Persistent Volumes because they are tied to the Pod lifecycle.

---

# 24. Stateful Application Storage

Example:

```
MySQL

↓

PVC

↓

Persistent Volume

↓

Cloud Disk
```

If the Pod is recreated:

```
New Pod

↓

Same PVC

↓

Same Volume

↓

Database Intact
```

---

## Why PVC Matters

Without persistent storage:

```
Database

↓

Pod Deleted

↓

Database Files Lost
```

With persistent storage:

```
Database

↓

PVC

↓

Persistent Volume

↓

Data Preserved
```

---

# Common Stateful Applications

- MySQL
- PostgreSQL
- MongoDB
- Redis (persistent mode)
- Kafka
- Elasticsearch
- Jenkins
- SonarQube
- GitLab
- Nexus Repository

---

# 25. Storage Performance Best Practices

### Choose the Right Storage

Examples:

- SSD for databases
- Shared file storage for shared content
- Object storage for backups (outside PV/PVC)

---

### Avoid hostPath

For production application data,

prefer CSI-backed storage over `hostPath`.

---

### Use Dynamic Provisioning

Preferred over manually creating PVs.

---

### Enable Monitoring

Track:

- Disk usage
- Latency
- IOPS
- Throughput
- Volume health

Common tools:

- Prometheus
- Grafana

---

### Plan Capacity

Monitor free space before applications reach storage limits.

---

### Back Up Data

Persistent volumes are **not** backups.

Use:

- CSI snapshots
- Backup software
- Cloud-native backup services

---

# 26. Storage Troubleshooting

## Step 1

Check PVC.

```bash
kubectl get pvc
```

Status:

```
Bound

Pending

Lost
```

---

## Step 2

Describe PVC.

```bash
kubectl describe pvc pvc-demo
```

Look for:

- Events
- StorageClass
- Access mode
- Requested capacity

---

## Step 3

Check PV.

```bash
kubectl get pv
```

---

## Step 4

Describe PV.

```bash
kubectl describe pv pv-demo
```

---

## Step 5

Verify StorageClass.

```bash
kubectl get storageclass
```

Check:

- Default StorageClass
- Provisioner
- Parameters

---

## Step 6

Check CSI Driver

```bash
kubectl get csidrivers
```

Verify that the required CSI driver is installed and healthy.

---

# Common Problems

| Problem | Possible Cause | Solution |
|---------|----------------|----------|
| PVC Pending | No matching PV or provisioning failure | Check StorageClass and CSI |
| Volume Mount Failed | Wrong access mode or attachment issue | Review Pod events |
| No Dynamic Provisioning | Missing/default StorageClass | Install or configure StorageClass |
| Expansion Failed | Driver doesn't support expansion | Verify CSI capabilities |
| Snapshot Failed | Snapshot CRDs/controller missing | Install CSI snapshot components |

---

# 27. Hands-on Labs

## Lab 1 - Create a PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: demo-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

Apply:

```bash
kubectl apply -f pvc.yaml
```

---

## Lab 2 - Use PVC in a Pod

Mount:

```yaml
volumes:
- name: storage
  persistentVolumeClaim:
    claimName: demo-pvc
```

Verify:

```bash
kubectl exec -it POD_NAME -- df -h
```

---

## Lab 3 - Expand the PVC

Increase:

```
5Gi

↓

10Gi
```

Apply the updated manifest and verify the PVC reflects the new size.

---

## Lab 4 - Observe Dynamic Provisioning

Create a PVC referencing the default StorageClass.

Watch:

```bash
kubectl get pvc,pv -w
```

Observe the PV being created and bound automatically.

---

## Lab 5 - Delete the Pod

Delete the Pod using the PVC.

Create a new Pod using the same PVC.

Verify:

- Data remains available.
- The PVC remains bound.

---

# 28. CKA Exam Tips

✔ Understand:

- PV
- PVC
- StorageClass
- CSI

✔ Know the difference between:

- Static provisioning
- Dynamic provisioning

✔ Memorize access modes:

- ReadWriteOnce
- ReadOnlyMany
- ReadWriteMany
- ReadWriteOncePod

✔ Practice:

```bash
kubectl get pv
kubectl get pvc
kubectl describe pvc
```

✔ Learn to identify why a PVC is stuck in `Pending`.

---

# 29. Interview Questions

## Beginner

1. What is a Persistent Volume?
2. What is a Persistent Volume Claim?
3. Why use a PVC instead of directly using a PV?
4. What is a StorageClass?
5. What is dynamic provisioning?

---

## Intermediate

1. Explain CSI.
2. Difference between RWO and RWX.
3. What is `WaitForFirstConsumer`?
4. Explain reclaim policies.
5. How does PVC binding work?

---

## Advanced

1. How would you design storage for a production PostgreSQL cluster?
2. Explain CSI volume expansion.
3. How would you troubleshoot a PVC stuck in `Pending`?
4. Explain storage topology awareness.
5. When would you use snapshots instead of backups?

---

## Scenario-Based

### Scenario 1

A PVC remains in `Pending`.

What would you check first?

---

### Scenario 2

A database Pod is recreated.

How do you ensure the data is preserved?

---

### Scenario 3

A team requests shared storage for multiple Pods writing simultaneously.

Which access mode and storage backend would you consider?

---

### Scenario 4

A volume expansion request was accepted, but the application still reports the old filesystem size.

What additional checks would you perform?

---

### Scenario 5

Your storage team asks why Kubernetes uses CSI.

How would you explain its benefits?

---

# 30. Storage Cheat Sheet

List PVs:

```bash
kubectl get pv
```

List PVCs:

```bash
kubectl get pvc
```

Describe PVC:

```bash
kubectl describe pvc demo-pvc
```

List StorageClasses:

```bash
kubectl get storageclass
```

List CSI Drivers:

```bash
kubectl get csidrivers
```

Watch provisioning:

```bash
kubectl get pv,pvc -w
```

Restart a workload after storage changes (if required):

```bash
kubectl rollout restart deployment app
```

---

# 31. Key Takeaways

- Containers have ephemeral writable layers.
- Volumes provide storage to Pods.
- `emptyDir` is temporary.
- `hostPath` is node-specific and should be used carefully.
- Persistent Volumes outlive Pods.
- Pods use PVCs rather than accessing PVs directly.
- StorageClasses enable dynamic provisioning.
- CSI is the standard storage interface.
- Access modes define how volumes are mounted.
- Reclaim policies control storage lifecycle after release.
- Volume expansion and snapshots depend on CSI driver capabilities.
- Persistent volumes are not backups.

---

# 32. Chapter Summary

Congratulations! 🎉

You have completed **K8S-10 – Volumes & Persistent Storage**.

In this chapter, you learned:

- Stateless vs Stateful applications
- Container writable layers
- Volumes
- `emptyDir`
- `hostPath`
- Persistent Volumes (PV)
- Persistent Volume Claims (PVC)
- PV/PVC binding
- StorageClasses
- CSI
- Access modes
- Reclaim policies
- Volume expansion
- Volume snapshots
- Ephemeral CSI volumes
- Stateful application storage
- Storage troubleshooting
- Hands-on labs
- CKA preparation
- Interview questions

You now have a strong foundation in Kubernetes storage and are ready to move on to **Ingress & Ingress Controllers**, where you'll learn how applications are securely exposed to users.

