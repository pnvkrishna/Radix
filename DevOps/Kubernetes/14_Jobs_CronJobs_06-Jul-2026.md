# K8S-14 - Jobs & CronJobs (Part 1)

> **Class Date:** 06-Jul-2026
>
> **Module:** Jobs & CronJobs


---

# 📚 Table of Contents

1. Introduction
2. Why Jobs?
3. What is a Job?
4. Job Lifecycle
5. Job Controller
6. Job vs Deployment vs CronJob
7. First Job YAML
8. Important Job Fields
9. Part 1 Summary

---

# 1. Introduction

Most Kubernetes workloads are designed to run continuously.

Examples:

- Web servers
- APIs
- Databases
- Monitoring agents

These workloads should always remain available.

However, some workloads are designed to complete and exit.

Examples:

- Backup database
- Import CSV data
- Generate monthly reports
- Resize images
- Run machine learning inference
- Perform schema migration

For these workloads Kubernetes provides **Jobs**.

---

# 2. Why Jobs?

Imagine a Deployment running:

```
Backup Script

↓

Completed

↓

Container Exits

↓

Deployment Creates New Pod

↓

Runs Again

↓

Infinite Loop
```

This is not desirable.

A backup should execute once and finish successfully.

Jobs solve this problem.

---

# 3. What is a Job?

A **Job** is a Kubernetes workload that ensures a task completes successfully.

Unlike a Deployment:

- The Pod is expected to exit.
- Successful completion is the desired outcome.
- Failed Pods can be retried automatically.

---

## Characteristics

✔ Runs to completion

✔ Tracks successful execution

✔ Supports retries

✔ Can run one or many Pods

✔ Suitable for batch processing

---

# 4. Job Lifecycle

```
Job Created

↓

Pod Created

↓

Container Starts

↓

Task Executes

↓

Success

↓

Pod Completes

↓

Job Marked Complete
```

---

## Failure Flow

```
Job Created

↓

Pod Starts

↓

Script Fails

↓

Pod Fails

↓

Retry (if allowed)

↓

Success OR Failure Limit Reached
```

---

# 5. Job Controller

The Job Controller continuously watches Job resources.

Responsibilities:

- Create Pods
- Monitor execution
- Retry failures
- Mark Jobs complete
- Mark Jobs failed after retry limits

---

## Architecture

```
        Job

         │

         ▼

   Job Controller

         │

         ▼

       Pod

         │

         ▼

 Container Executes

         │

 ┌───────┴────────┐

 ▼                ▼

Success         Failure

 │                │

 ▼                ▼

Complete      Retry (if allowed)
```

---

# 6. Job vs Deployment vs CronJob

| Feature | Deployment | Job | CronJob |
|----------|------------|-----|----------|
| Runs continuously | ✅ | ❌ | ❌ |
| Expected to exit | ❌ | ✅ | ✅ |
| Scheduled execution | ❌ | ❌ | ✅ |
| Retry failed Pods | ✅ | ✅ | ✅ |
| Typical use | Web/API | Batch task | Scheduled task |

---

# 7. First Job YAML

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: hello-job

spec:
  template:

    spec:

      restartPolicy: Never

      containers:

      - name: hello

        image: busybox:1.36

        command:

        - sh

        - -c

        - echo "Hello Kubernetes Job!"
```

---

## Apply

```bash
kubectl apply -f job.yaml
```

---

## Verify

```bash
kubectl get jobs
```

Example:

```
NAME         COMPLETIONS   DURATION   AGE

hello-job    1/1           5s         12s
```

---

Check Pods:

```bash
kubectl get pods
```

Possible output:

```
hello-job-x8k7m

Completed
```

---

View logs:

```bash
kubectl logs job/hello-job
```

Output:

```
Hello Kubernetes Job!
```

---

# 8. Important Job Fields

## restartPolicy

```yaml
restartPolicy: Never
```

or

```yaml
restartPolicy: OnFailure
```

For Jobs, only these restart policies are supported.

---

### Never

If the container exits:

```
Pod Ends

↓

Controller Decides Whether to Create Another Pod
```

---

### OnFailure

If the container exits with a failure code:

```
Container Fails

↓

Container Restarted Inside Same Pod

↓

If Pod Ultimately Fails

↓

Job Controller May Create Another Pod
```

This reduces unnecessary Pod creation for transient failures.

---

## template

Defines the Pod that the Job will run.

---

## containers

Defines the workload executed by the Job.

---

## command

Specifies the process to execute instead of the image's default entrypoint.

Example:

```yaml
command:

- python

- backup.py
```

---

# Beginner Example

```
Database

↓

Backup Script

↓

Job

↓

Backup File Created

↓

Job Complete
```

---

# Production Examples

- Nightly database backup
- One-time schema migration
- Data import
- CSV processing
- AI batch inference
- Video transcoding
- Image thumbnail generation
- Batch email generation

---

# 9. Part 1 Summary

In this section, you learned:

- Why Jobs exist
- Difference between Jobs and Deployments
- Job lifecycle
- Job controller
- Basic Job YAML
- restartPolicy
- Common production use cases

You now understand how Kubernetes executes one-time batch workloads reliably.

# K8S-14 - Jobs & CronJobs (Part 2)

> **Class Date:** 06-Jul-2026

---

# 📚 Table of Contents

10. Completions
11. Parallelism
12. Completion Modes
13. Indexed Jobs
14. Retry Policy (backoffLimit)
15. Active Deadline
16. TTL After Finished
17. Suspend & Resume
18. Production Examples
19. Complete Production YAML
20. Part 2 Summary

---

# 10. Completions

The `completions` field specifies **how many successful Pod executions** are required before the Job is considered complete.

Example:

```yaml
spec:
  completions: 5
```

The Job finishes only after **5 successful completions**.

---

## Example Flow

```
Job

↓

5 Successful Runs Required

↓

Pod-1 ✅

Pod-2 ✅

Pod-3 ✅

Pod-4 ✅

Pod-5 ✅

↓

Job Complete
```

---

## Default Behavior

If `completions` is omitted:

```
1 successful completion

↓

Job Complete
```

---

# 11. Parallelism

`parallelism` controls **how many Pods may run at the same time**.

Example:

```yaml
spec:
  parallelism: 3
  completions: 10
```

Execution:

```
10 Tasks

↓

3 Pods Running Together

↓

As Pods Finish

↓

New Pods Start

↓

Until 10 Completions
```

---

## Architecture

```
Job

↓

Parallelism = 3

↓

Pod-1

Pod-2

Pod-3

↓

Complete

↓

Next Pods Created
```

---

## Why Use Parallelism?

Useful for:

- Large CSV processing
- Image processing
- AI inference
- ETL jobs
- Data conversion

---

# 12. Completion Modes

Kubernetes supports two completion modes.

---

## NonIndexed (Default)

Pods are interchangeable.

```
Task

↓

Any Pod

↓

Success
```

No Pod has a unique identity.

---

## Indexed

Each Pod receives a unique completion index.

Example:

```
Worker-0

↓

Index 0

----------------

Worker-1

↓

Index 1

----------------

Worker-2

↓

Index 2
```

YAML:

```yaml
completionMode: Indexed
```

Kubernetes exposes the index to the Pod (for example, through annotations, labels, hostname patterns, and environment support depending on workload design), allowing each worker to process a distinct partition of work.

---

# 13. Indexed Jobs

Indexed Jobs are ideal when each worker is responsible for a different portion of the workload.

Example:

```
100 Files

↓

10 Workers

↓

Worker 0

Files 1-10

----------------

Worker 1

Files 11-20

----------------

Worker 2

Files 21-30
```

Each worker can determine which partition it owns by using its completion index.

---

## Production Use Cases

- Distributed ETL
- Machine Learning preprocessing
- Video rendering
- Scientific computing
- Batch simulations

---

# 14. Retry Policy (backoffLimit)

Jobs automatically retry failed executions.

Example:

```yaml
backoffLimit: 4
```

Meaning:

```
Failure

↓

Retry 1

↓

Retry 2

↓

Retry 3

↓

Retry 4

↓

Mark Job Failed
```

---

## Why?

Temporary failures can occur because of:

- Network interruptions
- External APIs
- Database startup delays
- Storage availability

Retries improve reliability without requiring manual intervention.

---

# 15. Active Deadline

Limit the total execution time.

Example:

```yaml
activeDeadlineSeconds: 600
```

Meaning:

```
Job Starts

↓

Runs

↓

10 Minutes Reached

↓

Job Terminated
```

Useful for preventing runaway Jobs.

---

# 16. TTL After Finished

Completed Jobs do not have to remain forever.

Example:

```yaml
ttlSecondsAfterFinished: 300
```

Meaning:

```
Job Complete

↓

Wait 5 Minutes

↓

Job Automatically Cleaned Up
```

This helps reduce API object clutter in busy clusters.

---

# 17. Suspend & Resume

A Job can be created in a suspended state.

Example:

```yaml
spec:
  suspend: true
```

Behavior:

```
Job Created

↓

No Pods Started
```

Resume:

```yaml
spec:
  suspend: false
```

Pods are then scheduled normally.

Useful for controlled execution windows.

---

# 18. Production Examples

## Database Migration

```
Application

↓

Migration Job

↓

Schema Updated

↓

Job Complete
```

---

## CSV Import

```
CSV File

↓

Job

↓

Database
```

---

## AI Batch Inference

```
Images

↓

Parallel Job

↓

Predictions

↓

Results Stored
```

---

## Video Processing

```
Videos

↓

Indexed Job

↓

Multiple Workers

↓

Rendered Videos
```

---

## Data Warehouse ETL

```
Source Database

↓

Parallel Jobs

↓

Transformation

↓

Data Warehouse
```

---

# 19. Complete Production YAML

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: image-processing

spec:

  completions: 20

  parallelism: 5

  completionMode: Indexed

  backoffLimit: 3

  activeDeadlineSeconds: 1800

  ttlSecondsAfterFinished: 600

  template:

    spec:

      restartPolicy: OnFailure

      containers:

      - name: worker

        image: python:3.12

        command:

        - python

        - process.py
```

---

# YAML Explained

## completions

Total successful executions required.

---

## parallelism

Maximum number of Pods running simultaneously.

---

## completionMode

Controls whether Pods are interchangeable (`NonIndexed`) or assigned unique indexes (`Indexed`).

---

## backoffLimit

Maximum retry attempts before the Job is marked as failed.

---

## activeDeadlineSeconds

Maximum runtime allowed for the Job.

---

## ttlSecondsAfterFinished

Automatically cleans up finished Jobs after the specified number of seconds.

---

## restartPolicy

For Jobs, use:

```yaml
Never
```

or

```yaml
OnFailure
```

---

# 20. Part 2 Summary

You learned:

- `completions`
- `parallelism`
- `completionMode`
- Indexed Jobs
- `backoffLimit`
- `activeDeadlineSeconds`
- `ttlSecondsAfterFinished`
- Suspend and resume
- Production batch-processing patterns
- Complete production Job YAML

You now understand how Kubernetes manages reliable, scalable, and parallel batch workloads.

# K8S-14 - Jobs & CronJobs (Part 3)

> **Class Date:** 06-Jul-2026
>
> **File:** `K8S-14_Jobs_CronJobs_06-Jul-2026.md`

---

# 📚 Table of Contents

21. What is a CronJob?
22. CronJob Lifecycle
23. Cron Schedule Syntax
24. Concurrency Policies
25. Missed Schedules
26. Time Zones
27. History Limits
28. Production Examples
29. Troubleshooting
30. Hands-on Labs
31. CKA Tips
32. Interview Questions
33. Cheat Sheet
34. Chapter Summary

---

# 21. What is a CronJob?

A **CronJob** creates **Jobs** according to a schedule.

Think of the relationship like this:

```
CronJob

↓

Creates

↓

Job

↓

Creates

↓

Pod

↓

Runs Task
```

A CronJob **does not run Pods directly**.

It creates a new Job each time the schedule is triggered.

---

## Real-world Examples

- Nightly database backup
- Weekly report generation
- Daily log cleanup
- Cache refresh
- Monthly billing
- Data synchronization
- Security scanning
- Certificate renewal

---

# 22. CronJob Lifecycle

```
Cron Schedule

↓

CronJob Triggered

↓

Job Created

↓

Pod Created

↓

Task Executes

↓

Success

↓

Job Complete
```

Next scheduled time:

```
CronJob

↓

Creates New Job

↓

Runs Again
```

---

# 23. Cron Schedule Syntax

Cron format consists of **five fields**:

```
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of Week (0-7)
│ │ │ └──── Month (1-12)
│ │ └────── Day of Month (1-31)
│ └──────── Hour (0-23)
└────────── Minute (0-59)
```

---

## Examples

### Every minute

```text
* * * * *
```

---

### Every day at midnight

```text
0 0 * * *
```

---

### Every day at 2:30 AM

```text
30 2 * * *
```

---

### Every Sunday at 1 AM

```text
0 1 * * 0
```

---

### Every 10 minutes

```text
*/10 * * * *
```

---

### Weekdays at 9 AM

```text
0 9 * * 1-5
```

---

# Example YAML

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: database-backup

spec:
  schedule: "0 2 * * *"

  jobTemplate:

    spec:

      template:

        spec:

          restartPolicy: OnFailure

          containers:

          - name: backup

            image: busybox:1.36

            command:

            - sh

            - -c

            - echo "Running Backup"
```

---

# 24. Concurrency Policies

Sometimes a previous Job is still running when the next schedule arrives.

Kubernetes provides three concurrency policies.

---

## Allow (Default)

```
2:00 AM

↓

Job-1 Running

↓

2:05 AM

↓

Job-2 Starts

```

Multiple Jobs can run simultaneously.

---

## Forbid

```
2:00 AM

↓

Job-1 Running

↓

2:05 AM

↓

Skip New Job
```

Use when overlapping executions are unsafe.

---

## Replace

```
2:00 AM

↓

Job-1 Running

↓

2:05 AM

↓

Stop Job-1

↓

Start Job-2
```

Useful when only the latest execution matters.

---

YAML:

```yaml
concurrencyPolicy: Forbid
```

---

# 25. Missed Schedules

If the controller misses a schedule (for example, due to downtime), Kubernetes can decide whether the missed execution is still valid.

Example:

```yaml
startingDeadlineSeconds: 300
```

Meaning:

```
Scheduled Time

↓

Missed

↓

Within 5 Minutes

↓

Job Still Starts

Otherwise

↓

Skipped
```

---

# 26. Time Zones

Modern Kubernetes supports specifying a time zone for CronJobs.

Example:

```yaml
timeZone: "Asia/Kolkata"
```

Without `timeZone`, the schedule is interpreted using the time zone configured for the Kubernetes control plane.

Using an explicit time zone helps avoid confusion, especially in multi-region environments.

---

# 27. History Limits

Completed Jobs do not need to be kept forever.

Limit retained history.

Example:

```yaml
successfulJobsHistoryLimit: 3

failedJobsHistoryLimit: 1
```

Behavior:

```
10 Successful Jobs

↓

Keep Latest 3

↓

Older Jobs Removed
```

---

# 28. Production Examples

## Database Backup

```
2:00 AM

↓

CronJob

↓

Backup

↓

Object Storage
```

---

## Daily Report

```
Midnight

↓

Generate Report

↓

Email Report
```

---

## ETL Pipeline

```
Every Hour

↓

Extract

↓

Transform

↓

Load
```

---

## Cache Cleanup

```
Every Night

↓

Delete Expired Cache

↓

Complete
```

---

## Security Scan

```
Weekly

↓

Scan Images

↓

Generate Findings

↓

Store Report
```

---

# 29. Troubleshooting

## List CronJobs

```bash
kubectl get cronjobs
```

---

## Describe CronJob

```bash
kubectl describe cronjob database-backup
```

---

## List Jobs

```bash
kubectl get jobs
```

---

## List Pods

```bash
kubectl get pods
```

---

## View Logs

```bash
kubectl logs job/<job-name>
```

---

## Common Problems

| Problem | Cause | Solution |
|----------|-------|----------|
| CronJob not running | Incorrect schedule | Verify cron expression |
| Jobs overlap | Default `Allow` policy | Use `Forbid` or `Replace` |
| Old Jobs accumulate | History limits not configured | Set history limits or TTL |
| Job never finishes | Application issue | Check logs and events |
| CronJob suspended | `spec.suspend: true` | Set `false` to resume |

---

# 30. Hands-on Labs

## Lab 1

Create a CronJob that runs every minute.

Verify:

```bash
kubectl get cronjobs
kubectl get jobs --watch
```

---

## Lab 2

Change schedule:

```
Every Minute

↓

Every 5 Minutes
```

Observe the new execution interval.

---

## Lab 3

Set:

```yaml
concurrencyPolicy: Forbid
```

Run a long-running Job and verify overlapping executions are skipped.

---

## Lab 4

Configure:

```yaml
successfulJobsHistoryLimit: 2
```

Observe automatic cleanup of older successful Jobs.

---

## Lab 5

Suspend the CronJob:

```yaml
suspend: true
```

Resume it later and verify new Jobs are created.

---

# 31. CKA Exam Tips

✔ Know the relationship:

```
CronJob

↓

Job

↓

Pod
```

✔ Memorize:

- `schedule`
- `jobTemplate`
- `concurrencyPolicy`
- `startingDeadlineSeconds`
- `successfulJobsHistoryLimit`
- `failedJobsHistoryLimit`
- `timeZone`

✔ Practice:

```bash
kubectl get cronjobs
kubectl create cronjob
kubectl describe cronjob
```

---

# 32. Interview Questions

## Beginner

1. What is a CronJob?
2. How is it different from a Job?
3. What resource does a CronJob create?
4. Explain cron syntax.
5. What is `jobTemplate`?

---

## Intermediate

1. Explain concurrency policies.
2. What happens if a schedule is missed?
3. Why configure history limits?
4. What does `timeZone` do?
5. How do you suspend a CronJob?

---

## Advanced

1. Design a nightly backup solution using CronJobs.
2. How would you prevent overlapping ETL executions?
3. Explain how CronJobs recover after controller downtime.
4. How would you troubleshoot missing scheduled Jobs?
5. When would you use `Replace` instead of `Forbid`?

---

# 33. Cheat Sheet

Create:

```bash
kubectl apply -f cronjob.yaml
```

List:

```bash
kubectl get cronjobs
```

Describe:

```bash
kubectl describe cronjob database-backup
```

Suspend:

```bash
kubectl patch cronjob database-backup \
  -p '{"spec":{"suspend":true}}'
```

Resume:

```bash
kubectl patch cronjob database-backup \
  -p '{"spec":{"suspend":false}}'
```

Delete:

```bash
kubectl delete cronjob database-backup
```

---

# 34. Chapter Summary

Congratulations! 🎉

You have completed **K8S-14 – Jobs & CronJobs**.

In this chapter, you learned:

- Jobs and batch processing
- Job lifecycle
- Parallel Jobs
- Indexed Jobs
- Retry behavior
- Active deadlines
- TTL cleanup
- CronJobs
- Cron expressions
- Concurrency policies
- Missed schedules
- Time zones
- History limits
- Troubleshooting
- Hands-on labs
- CKA preparation
- Interview questions

You now understand how Kubernetes executes one-time and scheduled workloads reliably in production.