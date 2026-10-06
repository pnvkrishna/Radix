# K8S-28 - Backup & Disaster Recovery (Part 1)

> **Class Date:** 20-Jul-2026
>
> **Module:** Kubernetes Backup & Disaster Recovery
>
> **File Name:** `K8S-28_Backup_Disaster_Recovery_20-Jul-2026.md`

---

# 📚 Table of Contents

1. Introduction
2. Why Backup & Disaster Recovery?
3. What Should Be Backed Up?
4. Backup Strategies
5. RPO & RTO
6. etcd Overview
7. etcd Backup
8. etcd Restore Basics
9. Part 1 Summary

---

# 1. Introduction

Failures are inevitable.

Examples:

- Accidental deletion
- Node failures
- Storage corruption
- Cloud outages
- Ransomware
- Human error
- Failed upgrades

The objective is not to prevent every failure—it is to **recover quickly and safely**.

---

# 2. Why Backup & Disaster Recovery?

Without backups:

```
Production Cluster

↓

Failure

↓

Data Lost
```

With backups:

```
Production Cluster

↓

Scheduled Backup

↓

Failure

↓

Restore

↓

Business Continues
```

---

# 3. What Should Be Backed Up?

Not everything has the same recovery requirements.

| Component | Backup? | Reason |
|-----------|----------|--------|
| etcd | ✅ Yes | Stores Kubernetes cluster state |
| Persistent Volumes | ✅ Yes | Application data |
| Custom Resources | ✅ Yes | Business configuration |
| Namespaces | ✅ Yes | Workload organization |
| Secrets | ✅ Yes | Credentials and certificates |
| ConfigMaps | ✅ Yes | Configuration |
| Container Images | Usually No | Rebuild or pull from registry if available |
| Node OS | Usually No | Nodes are generally replaceable |

---

## Categories

### Cluster State

- etcd
- CRDs
- RBAC
- Namespaces

---

### Application State

- Persistent Volumes
- Databases
- Object storage
- Message queues

---

### Configuration

- Helm values
- Git repositories
- GitOps manifests

---

# 4. Backup Strategies

---

## Full Backup

Everything is backed up.

```
Cluster

↓

Complete Backup
```

Advantages:

- Simple restore

Disadvantages:

- Larger storage
- Longer backup time

---

## Incremental Backup

Only changes since the previous backup.

```
Day 1

↓

Full

↓

Day 2

↓

Changes Only
```

Advantages:

- Faster
- Less storage

---

## Differential Backup

Changes since the last full backup.

```
Full

↓

Diff

↓

Diff
```

---

## Comparison

| Type | Storage | Restore Speed |
|------|----------|---------------|
| Full | High | Fast |
| Incremental | Low | Slower |
| Differential | Medium | Medium |

---

# 5. RPO & RTO

---

## Recovery Point Objective (RPO)

Maximum acceptable data loss.

Example:

```
RPO = 15 minutes
```

A backup every 15 minutes limits data loss to approximately 15 minutes.

---

## Recovery Time Objective (RTO)

Maximum acceptable recovery time.

Example:

```
RTO = 30 minutes
```

The system should be restored within 30 minutes.

---

## Relationship

```
Failure

↓

Recover

↓

RTO

↓

Recovered Data

↓

RPO
```

---

# 6. etcd Overview

`etcd` is Kubernetes' distributed key-value store.

It contains:

- Pods
- Deployments
- Services
- Secrets
- ConfigMaps
- CRDs
- RBAC
- Node information

If `etcd` is lost and no backup exists, recreating the cluster state may be extremely difficult.

> **Note:** etcd stores the Kubernetes API state. It does **not** store application data inside Persistent Volumes.

---

# 7. etcd Backup

The recommended backup method is `etcdctl snapshot save`.

Example:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
```

In secured environments, you'll also specify the endpoint and TLS certificates.

Example:

```bash
ETCDCTL_API=3 etcdctl \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save snapshot.db
```

---

## Verify Snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

Example output:

```
HASH
REVISION
TOTAL KEYS
TOTAL SIZE
```

Verifying backups ensures the snapshot is usable before an emergency.

---

# 8. etcd Restore Basics

Restore from a snapshot:

```bash
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
```

A restore creates a new data directory.

In a kubeadm cluster, additional steps typically include:

- Updating the etcd static Pod manifest (if restoring to a new data directory)
- Restarting the etcd static Pod
- Verifying API server connectivity

Always follow the recovery procedure for your Kubernetes distribution.

---

## Verify Cluster

After recovery:

```bash
kubectl get nodes
kubectl get pods -A
```

Confirm:

- API server is available
- Workloads are visible
- Cluster state matches expectations

---

# Backup Workflow

```
etcd

↓

Snapshot

↓

Secure Storage

↓

Failure

↓

Restore

↓

Cluster Available
```

---

# 9. Part 1 Summary

You learned:

- Why backups matter
- What should be backed up
- Backup strategies
- RPO
- RTO
- etcd overview
- etcd snapshot backup
- etcd restore basics

You now understand the foundation of Kubernetes backup and disaster recovery.

# K8S-28 – Backup & Disaster Recovery (Part 2)

> **Class Date:** 20-Jul-2026
>
> **Module:** Kubernetes Backup & Disaster Recovery

---

# 📚 Table of Contents

10. Velero Overview
11. Velero Architecture
12. Installing Velero
13. Creating Backups
14. Scheduled Backups
15. Restoring Resources
16. CSI VolumeSnapshots
17. Cross-Cluster Migration
18. Backup Verification
19. Disaster Recovery Architecture
20. Part 2 Summary

---

# 10. Velero Overview

Velero is an open-source Kubernetes backup and recovery tool.

It can back up:

- Kubernetes API resources
- Namespaces
- Custom Resources (CRDs)
- Persistent Volumes (when supported)
- Entire applications

---

## High-Level Architecture

```
                 Kubernetes Cluster
                        │
                        ▼
                  Velero Server
                        │
         ┌──────────────┴──────────────┐
         ▼                             ▼
 Kubernetes Resources          Volume Data
         │                             │
         ▼                             ▼
 Backup Metadata             CSI Snapshot / File-level Backup
         │                             │
         └──────────────┬──────────────┘
                        ▼
               Object Storage Bucket
         (S3 / Azure Blob / GCS / Compatible)
```

---

# 11. Velero Architecture

Main Components:

| Component | Purpose |
|-----------|----------|
| Velero Server | Coordinates backups and restores |
| Velero CLI | User interface |
| Object Storage | Stores backup metadata |
| CSI Plugin | Uses CSI snapshots where supported |
| Node Agent (optional) | File-system backup for unsupported volumes |

---

## Backup Flow

```
User

↓

Velero CLI

↓

Velero Server

↓

API Server

↓

Collect Resources

↓

Snapshot Volumes (optional)

↓

Upload Metadata

↓

Object Storage
```

---

# 12. Installing Velero

General workflow:

1. Install Velero CLI
2. Configure cloud credentials
3. Install Velero components
4. Verify installation

---

Example (AWS-style object storage):

```bash
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.x \
  --bucket my-backup-bucket \
  --backup-location-config region=ap-south-1
```

> Replace provider, plugin, bucket, and configuration based on your cloud platform.

---

## Verify Installation

```bash
kubectl get pods -n velero
```

Expected:

```
velero

Running
```

---

# 13. Creating Backups

Backup a namespace:

```bash
velero backup create orders-backup \
  --include-namespaces orders
```

---

Backup the entire cluster:

```bash
velero backup create full-cluster-backup
```

---

View backups:

```bash
velero backup get
```

---

Describe backup:

```bash
velero backup describe orders-backup
```

---

## Backup Lifecycle

```
Create Backup

↓

Collect Resources

↓

Snapshot Volumes (if configured)

↓

Upload

↓

Completed
```

---

# 14. Scheduled Backups

Daily backup example:

```bash
velero schedule create daily-backup \
  --schedule="0 2 * * *"
```

This creates a backup every day at **02:00**.

---

View schedules:

```bash
velero schedule get
```

---

Delete schedule:

```bash
velero schedule delete daily-backup
```

---

## Production Recommendations

Critical workloads:

- Frequent backups
- Longer retention
- Regular restore testing

Development workloads:

- Lower backup frequency
- Shorter retention

---

# 15. Restoring Resources

Restore from backup:

```bash
velero restore create \
  --from-backup orders-backup
```

---

List restores:

```bash
velero restore get
```

---

Describe restore:

```bash
velero restore describe <restore-name>
```

---

## Restore Workflow

```
Select Backup

↓

Download Metadata

↓

Recreate Resources

↓

Restore Volumes

↓

Verify Application
```

---

# 16. CSI VolumeSnapshots

Modern CSI drivers may support native volume snapshots.

```
Persistent Volume

↓

CSI Driver

↓

VolumeSnapshot

↓

Storage Snapshot
```

---

## Core Objects

| Resource | Purpose |
|----------|----------|
| VolumeSnapshot | Snapshot request |
| VolumeSnapshotContent | Actual snapshot object |
| VolumeSnapshotClass | Snapshot policy |

---

Example:

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot

metadata:
  name: db-snapshot

spec:
  source:
    persistentVolumeClaimName: db-pvc

  volumeSnapshotClassName: csi-snapclass
```

---

## Benefits

- Fast
- Storage-efficient
- Storage-provider integration
- Crash-consistent snapshots (behavior depends on the storage system)

---

# 17. Cross-Cluster Migration

Velero can help migrate workloads.

```
Cluster A

↓

Backup

↓

Object Storage

↓

Restore

↓

Cluster B
```

Common scenarios:

- Cloud migration
- Region migration
- Kubernetes upgrades
- Disaster recovery

> Successful migration depends on compatibility between Kubernetes versions, storage classes, CRDs, and cloud resources.

---

# 18. Backup Verification

A backup is not complete until it has been tested.

---

## Verify

Check status:

```bash
velero backup get
```

---

Inspect details:

```bash
velero backup describe orders-backup
```

---

Test restore:

- Restore into a non-production cluster
- Validate workloads
- Confirm application functionality
- Verify Persistent Volume recovery

---

## Validation Checklist

- Pods Running
- Services Reachable
- Data Present
- Secrets Restored
- ConfigMaps Correct
- Ingress Functional

---

# 19. Disaster Recovery Architecture

```
                Production Cluster
                        │
                        ▼
                Scheduled Backups
                        │
                        ▼
              Object Storage Repository
                        │
         ┌──────────────┴──────────────┐
         ▼                             ▼
  Secondary Cluster             Restore Testing
         │                             │
         ▼                             ▼
 Business Continuity          Backup Validation
```

---

## Production Best Practices

- Automate backups
- Encrypt backup storage
- Use immutable storage where supported
- Replicate backups across regions
- Monitor backup failures
- Test restores regularly
- Protect backup credentials with least privilege

---

# 20. Part 2 Summary

You learned:

- Velero architecture
- Installation workflow
- Creating backups
- Scheduled backups
- Restoring resources
- CSI VolumeSnapshots
- Cross-cluster migration
- Backup verification
- Disaster recovery architecture

You now understand how production Kubernetes environments automate backup and restore operations.

# K8S-28 – Backup & Disaster Recovery (Part 3)

> **Class Date:** 20-Jul-2026
>
> **Module:** Kubernetes Backup & Disaster Recovery

---

# 📚 Table of Contents

21. High Availability vs Disaster Recovery
22. Multi-Cluster DR Strategies
23. etcd Disaster Recovery Workflow
24. Backup Security
25. Backup Retention Policies
26. Disaster Recovery Runbooks
27. Recovery Testing
28. Real Production Case Studies
29. Hands-on Labs
30. CKA / CKAD / CKS Interview Questions
31. Backup & DR Cheat Sheet
32. Chapter Summary

---

# 21. High Availability vs Disaster Recovery

These concepts complement each other but solve different problems.

| High Availability (HA) | Disaster Recovery (DR) |
|-------------------------|------------------------|
| Reduces downtime | Recovers after major failures |
| Handles component failures | Handles catastrophic events |
| Automatic failover | Planned restoration |
| Seconds to minutes | Minutes to hours (or longer) |

---

## High Availability

Example:

```
            Load Balancer
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
 API Server   API Server   API Server
      │           │           │
      └───────────┼───────────┘
                  ▼
          Highly Available etcd
```

A single node failure should not interrupt cluster operations.

---

## Disaster Recovery

```
Primary Cluster

↓

Regional Failure

↓

Restore

↓

Secondary Cluster

↓

Application Available
```

HA helps **avoid outages**.

DR helps **recover from outages that HA cannot prevent**.

---

# 22. Multi-Cluster DR Strategies

## Active–Passive

```
Users

↓

Primary Cluster

↓

Scheduled Replication

↓

Standby Cluster
```

Advantages:

- Lower cost
- Simpler operations

Disadvantages:

- Recovery takes longer
- Standby cluster must be kept current

---

## Active–Active

```
             Global Load Balancer
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
   Cluster A               Cluster B
```

Advantages:

- High availability
- Geographic resilience

Disadvantages:

- Greater complexity
- Data synchronization challenges

---

## Backup Repository

```
Cluster A

↓

Backups

↓

Object Storage

↓

Restore Anywhere
```

Store backups independently of the cluster.

---

# 23. etcd Disaster Recovery Workflow

## Before Failure

```
Scheduled Snapshot

↓

Encrypted Storage

↓

Integrity Verification
```

---

## During Recovery

```
Control Plane Offline

↓

Restore Snapshot

↓

Rebuild etcd

↓

Restart Control Plane

↓

Validate Cluster
```

---

## Validation Checklist

After restore:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get deployments -A
kubectl get pvc
```

Confirm:

- Nodes Ready
- Core components healthy
- Workloads restored
- Storage available

---

# 24. Backup Security

Backups contain sensitive information.

Examples:

- Secrets
- Certificates
- ServiceAccount tokens
- Configuration
- Business metadata

Treat backups as production data.

---

## Best Practices

### Encrypt

Protect:

- Object storage
- Snapshot repositories
- Local backup files

---

### Least Privilege

Grant only the permissions required to:

- Create backups
- Read backups
- Restore backups

Separate operational roles when possible.

---

### Immutable Storage

Where supported:

```
Backup

↓

Write Once

↓

Cannot Be Modified
```

This reduces the impact of accidental deletion or ransomware.

---

### Audit

Log:

- Backup creation
- Restore operations
- Permission changes

---

# 25. Backup Retention Policies

Example policy:

| Backup Type | Frequency | Retention |
|-------------|-----------|-----------|
| Hourly | Every hour | 24 hours |
| Daily | Every day | 30 days |
| Weekly | Every week | 12 weeks |
| Monthly | Every month | 12 months |

Retention depends on:

- Compliance
- Business requirements
- Storage cost
- Recovery needs

---

# 26. Disaster Recovery Runbooks

A runbook provides documented recovery steps.

---

## Example Workflow

```
Alert

↓

Confirm Failure

↓

Assess Impact

↓

Select Backup

↓

Restore Cluster

↓

Restore Applications

↓

Validate

↓

Return Service
```

---

## Typical Runbook Sections

- Scope
- Prerequisites
- Recovery steps
- Validation
- Rollback plan
- Escalation contacts
- Lessons learned

Keep runbooks version-controlled and reviewed regularly.

---

# 27. Recovery Testing

Backups that are never tested should not be assumed to work.

---

## Recommended Exercises

- Restore a namespace
- Restore an application
- Restore Persistent Volumes
- Restore an etcd snapshot
- Simulate region loss (in a test environment)

---

## Recovery Metrics

Measure:

- Recovery Time (RTO)
- Data Loss (RPO)
- Success Rate
- Validation Time

Track improvements over time.

---

# 28. Real Production Case Studies

---

## Case Study 1 — Accidental Namespace Deletion

```
Developer Error

↓

Namespace Deleted

↓

Velero Restore

↓

Namespace Recovered
```

Key lesson:

Namespace-level backups reduce recovery time.

---

## Case Study 2 — etcd Corruption

```
etcd Failure

↓

API Server Unavailable

↓

Restore Snapshot

↓

Restart Control Plane

↓

Cluster Operational
```

Key lesson:

Regular snapshot verification is as important as snapshot creation.

---

## Case Study 3 — Regional Cloud Outage

```
Primary Region

↓

Unavailable

↓

Restore to Secondary Region

↓

DNS / Traffic Switch

↓

Users Connected
```

Key lesson:

Off-site backups and documented recovery procedures are essential.

---

## Case Study 4 — Ransomware

```
Compromised Cluster

↓

Isolate Environment

↓

Restore from Immutable Backup

↓

Rotate Credentials

↓

Resume Service
```

Key lesson:

Backups alone are insufficient—credential rotation and security review are also required.

---

# 29. Hands-on Labs

## Lab 1

Create an etcd snapshot.

Verify it using:

```bash
etcdctl snapshot status snapshot.db
```

---

## Lab 2

Install Velero in a test cluster.

Create:

- Namespace backup
- Scheduled backup

---

## Lab 3

Restore a namespace into a non-production cluster.

Validate:

- Pods
- Services
- Secrets
- ConfigMaps

---

## Lab 4

Create a CSI VolumeSnapshot (if supported by your storage driver).

Restore the associated Persistent Volume.

---

## Lab 5

Write a Disaster Recovery Runbook for:

- Cluster failure
- Namespace deletion
- Storage failure

Include validation steps and recovery criteria.

---

# 30. CKA / CKAD / CKS Interview Questions

> **Note:** Backup and disaster recovery concepts are valuable for all Kubernetes practitioners. Deep operational details (such as etcd recovery and DR planning) are especially relevant for platform engineering, SRE, and production operations.

## Beginner

1. What is disaster recovery?
2. What is RPO?
3. What is RTO?
4. Why should etcd be backed up?
5. What does Velero do?

---

## Intermediate

1. Explain CSI VolumeSnapshots.
2. How would you restore a namespace?
3. Why should restore testing be performed?
4. Explain backup retention policies.
5. Compare HA and DR.

---

## Advanced

1. Design a multi-region Kubernetes DR architecture.
2. How would you recover from etcd corruption?
3. How would you secure backup repositories?
4. Design backup policies for business-critical applications.
5. How would you validate a DR exercise?

---

## Scenario-Based

### Scenario 1

A production namespace is accidentally deleted.

How would you restore service with minimal downtime?

---

### Scenario 2

An object storage bucket containing backups becomes unavailable.

How would you reduce this single point of failure?

---

### Scenario 3

A cloud region experiences a prolonged outage.

How would your DR strategy change depending on whether the architecture is Active–Passive or Active–Active?

---

### Scenario 4

A restore completes successfully, but users still report failures.

What post-restore validation would you perform?

---

### Scenario 5

Security discovers that backup credentials were compromised.

What immediate response actions would you take?

---

# 31. Backup & DR Cheat Sheet

## etcd

Backup:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
```

Verify:

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

Restore:

```bash
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db
```

---

## Velero

Create backup:

```bash
velero backup create full-backup
```

List backups:

```bash
velero backup get
```

Create schedule:

```bash
velero schedule create daily-backup \
  --schedule="0 2 * * *"
```

Restore:

```bash
velero restore create \
  --from-backup full-backup
```

---

## Validation

```bash
kubectl get nodes
kubectl get pods -A
kubectl get pvc
kubectl get ingress
```

---

# 32. Chapter Summary

Congratulations! 🎉

You have completed **K8S-28 – Backup & Disaster Recovery**.

In this chapter, you learned:

- Backup fundamentals
- RPO & RTO
- etcd backup and restore
- Velero architecture
- Namespace backups
- Scheduled backups
- Restore workflows
- CSI VolumeSnapshots
- High Availability vs Disaster Recovery
- Multi-cluster recovery strategies
- Backup security
- Retention policies
- Recovery testing
- Disaster recovery runbooks
- Production case studies
- Hands-on labs
- Interview questions

You now have a strong foundation for designing, operating, and validating production-grade Kubernetes backup and disaster recovery solutions.

