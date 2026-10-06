# K8S-17 - Health Checks (Liveness, Readiness & Startup Probes) (Part 1)

> **Class Date:** 09-Jul-2026
>
> **Module:** Health Checks
>
> **File Name:** `K8S-17_Health_Checks_09-Jul-2026.md`

---

# 📚 Table of Contents

1. Introduction
2. Why Health Checks?
3. Types of Probes
4. Liveness Probe
5. Readiness Probe
6. Startup Probe
7. Probe Lifecycle
8. First Probe YAML
9. Part 1 Summary

---

# 1. Introduction

A running container is **not necessarily a healthy application**.

Example:

```
Container

↓

Running

↓

Application Thread Deadlocked

↓

No Responses
```

From Kubernetes' perspective:

```
Container = Running
```

From the user's perspective:

```
Application = Broken
```

Health probes help Kubernetes detect these situations automatically.

---

# 2. Why Health Checks?

Imagine an application:

```
Pod

↓

Container Running

↓

Database Connection Lost

↓

API Returns 500 Errors
```

Without probes:

```
Service

↓

Still Sends Traffic

↓

Users See Failures
```

With probes:

```
Readiness Probe Fails

↓

Pod Removed from Service Endpoints

↓

Traffic Sent to Healthy Pods
```

If the application becomes permanently unhealthy:

```
Liveness Probe Fails

↓

Container Restarted
```

---

# 3. Types of Probes

Kubernetes provides three probe types.

| Probe | Purpose | Action |
|--------|---------|--------|
| Liveness | Is the application still running correctly? | Restart container if unhealthy |
| Readiness | Can the application serve traffic? | Remove Pod from Service endpoints until ready |
| Startup | Has the application finished starting? | Delay liveness/readiness checks during startup |

---

## Probe Flow

```
Container Starts

↓

Startup Probe

↓

Startup Successful

↓

Readiness Probe

↓

Traffic Allowed

↓

Liveness Probe

↓

Restart if Necessary
```

---

# 4. Liveness Probe

The liveness probe answers:

> **"Should this container be restarted?"**

Example:

```
Application

↓

Deadlock

↓

No Responses

↓

Liveness Fails

↓

Container Restarted
```

---

## Common Use Cases

- Deadlocks
- Infinite loops
- Hung processes
- Unrecoverable internal errors

---

## Important

Use liveness probes only for failures that **cannot recover without a restart**.

Do **not** use liveness probes to detect temporary dependency failures (for example, a briefly unavailable external API).

---

# 5. Readiness Probe

The readiness probe answers:

> **"Can this Pod receive traffic right now?"**

Example:

```
Pod Starts

↓

Loading Cache

↓

Readiness = Failed

↓

No Traffic

↓

Initialization Complete

↓

Readiness = Success

↓

Traffic Begins
```

---

## Benefits

Readiness probes prevent:

- Failed user requests
- Traffic during initialization
- Traffic during graceful shutdown
- Traffic while dependencies are unavailable (if your application intentionally reports itself as not ready)

---

# 6. Startup Probe

Some applications need significant time to start.

Examples:

- Java applications
- Spring Boot
- Large machine learning models
- Elasticsearch
- Databases

Without a startup probe:

```
Application Starting

↓

Liveness Runs Too Early

↓

Fails

↓

Restart

↓

Infinite Restart Loop
```

---

With a startup probe:

```
Application Starts

↓

Startup Probe Waits

↓

Initialization Completes

↓

Readiness Begins

↓

Liveness Begins
```

---

# 7. Probe Lifecycle

Complete flow:

```
Container Starts

↓

Startup Probe

↓

Success

↓

Readiness Probe

↓

Ready

↓

Service Routes Traffic

↓

Liveness Probe

↓

Healthy

↓

Application Continues
```

If liveness later fails:

```
Liveness Failure

↓

Container Restarted

↓

Startup Sequence Begins Again (if a startup probe is configured)
```

---

# 8. First Probe YAML

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:

  containers:

  - name: nginx

    image: nginx:1.27

    ports:

    - containerPort: 80

    livenessProbe:

      httpGet:

        path: /

        port: 80

      initialDelaySeconds: 10

      periodSeconds: 10

    readinessProbe:

      httpGet:

        path: /

        port: 80

      initialDelaySeconds: 5

      periodSeconds: 5
```

---

# YAML Explained

## livenessProbe

Determines whether the container should be restarted.

---

## readinessProbe

Determines whether the Pod should receive traffic.

---

## initialDelaySeconds

Wait before the first probe after the container starts.

---

## periodSeconds

How often the probe runs.

---

## Verify

Create:

```bash
kubectl apply -f pod.yaml
```

Describe:

```bash
kubectl describe pod nginx
```

Review:

```
Liveness:

Readiness:
```

---

# Production Example

```
Client

↓

Service

↓

Ready Pods Only

↓

Application

↓

Liveness Monitors

↓

Restart if Permanently Unhealthy
```

---

# 9. Part 1 Summary

You learned:

- Why health checks are important
- Difference between running and healthy
- Liveness probe
- Readiness probe
- Startup probe
- Probe lifecycle
- Basic probe YAML

You now understand the purpose of Kubernetes health probes and when each type should be used.

# K8S-17 - Health Checks (Part 2)

> **Class Date:** 09-Jul-2026

---

# 📚 Table of Contents

10. HTTP Probes
11. TCP Socket Probes
12. Exec Probes
13. Probe Timing Parameters
14. Probe Result Thresholds
15. Probe Lifecycle Example
16. Production Probe YAML
17. Common Probe Patterns
18. Part 2 Summary

---

# 10. HTTP Probes

HTTP probes send an HTTP request to the application.

Typical endpoint:

```
GET /health
```

Example:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
```

---

## Architecture

```
Kubelet

↓

HTTP GET

↓

Application

↓

200 OK

↓

Healthy
```

If the endpoint returns an error (for example, HTTP 500) or cannot be reached:

```
Probe Failed
```

---

## Best Practice

Provide dedicated health endpoints such as:

```
/health
/live
/ready
```

Avoid using complex business APIs as health checks.

---

# 11. TCP Socket Probes

TCP probes only verify that a TCP connection can be established.

Example:

```yaml
readinessProbe:
  tcpSocket:
    port: 5432
```

---

## Architecture

```
Kubelet

↓

TCP Connect

↓

Port Open

↓

Success
```

If the connection cannot be established:

```
Failure
```

---

## Suitable For

- Databases
- Message brokers
- Simple TCP services

Remember: a TCP probe does **not** verify that the application is functioning correctly—it only confirms that the port accepts connections.

---

# 12. Exec Probes

Exec probes execute a command inside the container.

Example:

```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
```

If the command exits with status code `0`:

```
Healthy
```

Non-zero exit code:

```
Probe Failed
```

---

## Common Uses

- Verify a file exists.
- Check a process.
- Run lightweight internal diagnostics.

Keep commands fast and lightweight.

---

# 13. Probe Timing Parameters

Several parameters control probe behavior.

---

## initialDelaySeconds

Wait before running the first probe.

Example:

```yaml
initialDelaySeconds: 30
```

Useful for applications that need startup time.

---

## periodSeconds

How often Kubernetes runs the probe.

Example:

```yaml
periodSeconds: 10
```

Meaning:

```
Every 10 Seconds
```

---

## timeoutSeconds

Maximum time Kubernetes waits for a probe response.

Example:

```yaml
timeoutSeconds: 3
```

If the probe does not complete within 3 seconds:

```
Probe Failed
```

---

# 14. Probe Result Thresholds

## failureThreshold

Number of consecutive failures before Kubernetes treats the probe as failed.

Example:

```yaml
failureThreshold: 3
```

Flow:

```
Failure

↓

Failure

↓

Failure

↓

Action Taken
```

---

## successThreshold

Number of consecutive successes required before considering the probe successful again.

Example:

```yaml
successThreshold: 2
```

This is primarily useful for **readiness probes**.

For **liveness** and **startup** probes, Kubernetes requires `successThreshold` to be `1`.

---

# Timing Summary

| Parameter | Purpose |
|-----------|---------|
| initialDelaySeconds | Delay before first probe |
| periodSeconds | Probe interval |
| timeoutSeconds | Probe timeout |
| failureThreshold | Consecutive failures before action |
| successThreshold | Consecutive successes before recovery (readiness only) |

---

# 15. Probe Lifecycle Example

```
Container Starts

↓

initialDelaySeconds

↓

Startup Probe (if configured)

↓

Readiness Probe

↓

Service Adds Pod

↓

Liveness Probe

↓

Healthy

↓

Application Continues
```

If the liveness probe later fails repeatedly:

```
Failure Threshold Reached

↓

Container Restarted
```

---

# 16. Production Probe YAML

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: payment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: payment

  template:

    metadata:
      labels:
        app: payment

    spec:

      containers:

      - name: payment

        image: payment:v2

        ports:

        - containerPort: 8080

        startupProbe:

          httpGet:
            path: /health/startup
            port: 8080

          failureThreshold: 30
          periodSeconds: 10

        readinessProbe:

          httpGet:
            path: /health/ready
            port: 8080

          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 2
          failureThreshold: 3
          successThreshold: 2

        livenessProbe:

          httpGet:
            path: /health/live
            port: 8080

          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 2
          failureThreshold: 3
```

---

# YAML Explained

## startupProbe

Protects slow-starting applications from premature restarts.

---

## readinessProbe

Controls whether the Pod receives traffic.

---

## livenessProbe

Restarts the container if it becomes permanently unhealthy.

---

# 17. Common Probe Patterns

## Web Applications

- Startup: `/health/startup`
- Readiness: `/health/ready`
- Liveness: `/health/live`

---

## Databases

Often use:

```
TCP Probe
```

or a lightweight readiness check if the database supports one.

---

## Batch Jobs

Many Jobs do not require liveness or readiness probes because they perform work and exit.

Evaluate probe usage based on workload behavior.

---

## Message Consumers

Readiness may depend on:

- Broker connectivity
- Internal initialization
- Required subscriptions

---

# 18. Part 2 Summary

You learned:

- HTTP probes
- TCP probes
- Exec probes
- Timing parameters
- Threshold parameters
- Probe lifecycle
- Production Deployment YAML
- Common probe patterns

You now understand how Kubernetes evaluates application health and how probe configuration affects runtime behavior.

# K8S-17 - Health Checks (Part 3)

> **Class Date:** 09-Jul-2026

---

# 📚 Table of Contents

19. CrashLoopBackOff and Probes
20. Rolling Updates & Readiness
21. Graceful Shutdown
22. Common Probe Misconfigurations
23. Troubleshooting
24. Production Best Practices
25. Hands-on Labs
26. CKA Exam Tips
27. Interview Questions
28. Health Checks Cheat Sheet
29. Chapter Summary

---

# 19. CrashLoopBackOff and Probes

One of the most common production issues is a Pod repeatedly restarting because of a misconfigured liveness probe.

Example:

```
Application Startup Time

60 Seconds

↓

Liveness Probe Starts

After 10 Seconds

↓

Probe Fails

↓

Container Restarted

↓

Repeat

↓

CrashLoopBackOff
```

---

## Why It Happens

Common reasons:

- Startup takes longer than expected.
- Probe endpoint is incorrect.
- Timeout is too short.
- Probe checks an unhealthy dependency instead of the application itself.

---

## Solution

For slow-starting applications:

- Configure a `startupProbe`.
- Increase `initialDelaySeconds` if appropriate.
- Verify the health endpoint manually.

---

# 20. Rolling Updates & Readiness

Readiness probes are essential for zero-downtime deployments.

Deployment flow:

```
Old Pod

↓

Running

↓

New Pod Created

↓

Readiness Probe Fails

↓

No Traffic

↓

Initialization Completes

↓

Readiness Succeeds

↓

Service Routes Traffic

↓

Old Pod Removed
```

Without readiness probes:

```
New Pod

↓

Immediately Receives Traffic

↓

Application Not Ready

↓

User Errors
```

---

# 21. Graceful Shutdown

When a Pod is terminated:

```
kubectl delete pod

↓

SIGTERM Sent

↓

Application Begins Shutdown

↓

Readiness Probe Fails

↓

Service Stops Sending Traffic

↓

Existing Requests Finish

↓

Container Exits
```

---

## terminationGracePeriodSeconds

Example:

```yaml
spec:
  terminationGracePeriodSeconds: 30
```

This gives the application time to finish in-flight requests before being forcibly terminated.

---

## Best Practice

Applications should:

- Handle `SIGTERM`.
- Stop accepting new work.
- Complete existing work where possible.
- Exit before the grace period expires.

---

# 22. Common Probe Misconfigurations

## Same Endpoint for Everything

Avoid using:

```
/health
```

for:

- Startup
- Readiness
- Liveness

Prefer separate endpoints with distinct responsibilities.

---

## Very Aggressive Timeouts

Example:

```yaml
timeoutSeconds: 1
```

On a busy node, a healthy application may occasionally respond slower than one second.

Choose values based on realistic response times.

---

## Checking External Dependencies

Example:

```
Liveness

↓

Checks Remote Database

↓

Temporary Network Issue

↓

Container Restarted
```

This often makes the situation worse.

Liveness should primarily determine whether the application itself is recoverable through a restart.

---

## Heavy Probe Logic

Avoid probes that:

- Execute expensive SQL queries.
- Perform complex business logic.
- Call multiple downstream services.

Health checks should be lightweight.

---

# 23. Troubleshooting

## Describe the Pod

```bash
kubectl describe pod <pod-name>
```

Look for:

```
Liveness:

Readiness:

Startup:

Events:
```

---

## View Logs

Current container:

```bash
kubectl logs <pod-name>
```

Previous crashed container:

```bash
kubectl logs <pod-name> --previous
```

---

## Test the Endpoint

Port-forward:

```bash
kubectl port-forward pod/<pod-name> 8080:8080
```

Then verify:

```bash
curl http://localhost:8080/health/live
curl http://localhost:8080/health/ready
curl http://localhost:8080/health/startup
```

---

## Check Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Look for repeated probe failures or restart events.

---

# Common Problems

| Problem | Possible Cause | Solution |
|----------|----------------|----------|
| CrashLoopBackOff | Liveness probe too aggressive | Add `startupProbe`, adjust timing |
| Service returns errors after deployment | Readiness probe missing or incorrect | Configure readiness correctly |
| Pod never becomes Ready | Health endpoint always failing | Verify application logic and endpoint |
| Frequent restarts | Liveness endpoint returns failures | Inspect logs and validate probe configuration |
| Slow deployments | Readiness succeeds too late | Review initialization sequence and probe timing |

---

# 24. Production Best Practices

## Separate Endpoints

Example:

```
/health/live

/health/ready

/health/startup
```

Each endpoint should represent a different health state.

---

## Keep Probes Lightweight

Health checks should complete quickly.

Avoid unnecessary work.

---

## Use Startup Probes

Recommended for:

- Java applications
- Spring Boot
- Databases
- Elasticsearch
- Large AI/ML services

---

## Monitor Probe Failures

Use observability tools such as:

- Prometheus
- Grafana

to identify recurring probe failures before they become outages.

---

## Test During Failure Scenarios

Validate probe behavior during:

- Dependency outages
- Slow startups
- Rolling updates
- Graceful shutdown

---

# 25. Hands-on Labs

## Lab 1

Deploy an application with:

- Liveness probe
- Readiness probe

Observe Pod status:

```bash
kubectl get pods -w
```

---

## Lab 2

Configure an incorrect readiness endpoint.

Verify:

- Pod remains **NotReady**
- Service does not route traffic

---

## Lab 3

Configure an incorrect liveness endpoint.

Observe:

```
CrashLoopBackOff
```

Then fix the configuration.

---

## Lab 4

Deploy a slow-starting application.

Compare behavior:

- Without `startupProbe`
- With `startupProbe`

---

## Lab 5

Perform a rolling update.

Observe:

- New Pods become Ready.
- Traffic shifts only after readiness succeeds.

---

# 26. CKA Exam Tips

✔ Know the purpose of each probe.

✔ Remember:

- Liveness → restart.
- Readiness → traffic.
- Startup → initialization.

✔ Practice:

```bash
kubectl describe pod
kubectl logs
kubectl logs --previous
kubectl port-forward
```

✔ Be comfortable editing probe settings quickly during the exam.

---

# 27. Interview Questions

## Beginner

1. What is a liveness probe?
2. What is a readiness probe?
3. What is a startup probe?
4. Why is readiness important?
5. What happens when a liveness probe fails?

---

## Intermediate

1. Compare HTTP, TCP, and Exec probes.
2. Why should liveness avoid checking external dependencies?
3. Explain `initialDelaySeconds`.
4. What is `failureThreshold`?
5. How do probes support rolling updates?

---

## Advanced

1. Design health checks for a microservices application.
2. How would you troubleshoot repeated CrashLoopBackOff caused by probes?
3. Explain probe behavior during graceful shutdown.
4. Design probe endpoints for a payment API.
5. How would you configure probes for a slow-starting Java service?

---

## Scenario-Based

### Scenario 1

A Pod restarts every minute after deployment.

Which probe configuration would you inspect first?

---

### Scenario 2

A new deployment receives traffic before initialization completes.

Which probe is missing or misconfigured?

---

### Scenario 3

Users report intermittent failures during rolling updates.

How can readiness probes reduce these errors?

---

### Scenario 4

A Spring Boot application takes 90 seconds to start.

How would you configure startup, readiness, and liveness probes?

---

### Scenario 5

A health endpoint performs several expensive database queries.

Why is this a poor probe design?

---

# 28. Health Checks Cheat Sheet

Describe Pod:

```bash
kubectl describe pod <pod-name>
```

View logs:

```bash
kubectl logs <pod-name>
kubectl logs <pod-name> --previous
```

Watch Pod status:

```bash
kubectl get pods -w
```

Port-forward:

```bash
kubectl port-forward pod/<pod-name> 8080:8080
```

Test endpoints:

```bash
curl http://localhost:8080/health/live
curl http://localhost:8080/health/ready
curl http://localhost:8080/health/startup
```

Show events:

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

# 29. Chapter Summary

Congratulations! 🎉

You have completed **K8S-17 – Health Checks (Liveness, Readiness & Startup Probes)**.

In this chapter, you learned:

- Why health checks matter
- Liveness probes
- Readiness probes
- Startup probes
- HTTP, TCP, and Exec probes
- Probe timing parameters
- Threshold settings
- Rolling updates
- Graceful shutdown
- CrashLoopBackOff troubleshooting
- Production best practices
- Hands-on labs
- CKA preparation
- Interview questions

You now understand how Kubernetes determines application health, controls traffic, and performs self-healing while supporting reliable, zero-downtime deployments.