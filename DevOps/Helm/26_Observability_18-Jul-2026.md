# K8S-26 - Observability (Metrics, Logs & Traces) (Part 1)

> **Class Date:** 18-Jul-2026
>
> **Module:** Kubernetes Observability
>
> **File Name:** `K8S-26_Observability_18-Jul-2026.md`

---

# 📚 Table of Contents

1. Introduction
2. What is Observability?
3. Monitoring vs Observability
4. Three Pillars of Observability
5. Metrics
6. Logs
7. Traces
8. Observability Architecture
9. Metrics Server
10. Prometheus
11. kube-state-metrics
12. Node Exporter
13. Part 1 Summary

---

# 1. Introduction

Operating Kubernetes in production requires answering questions like:

- Is the cluster healthy?
- Why is a Pod restarting?
- Which node is under high CPU load?
- Why is the application slow?
- Which deployment introduced errors?

Observability provides the tools and data needed to answer these questions.

---

# 2. What is Observability?

Observability is the ability to understand the internal state of a system by examining the telemetry it produces.

Telemetry includes:

- Metrics
- Logs
- Traces

Unlike simple monitoring, observability helps explain **why** something happened, not just **that** it happened.

---

# 3. Monitoring vs Observability

| Monitoring | Observability |
|------------|---------------|
| Watches predefined metrics | Explores unknown issues |
| Detects known failures | Helps diagnose new failures |
| Focuses on alerts | Focuses on understanding system behavior |
| Answers "Is something wrong?" | Answers "Why is it wrong?" |

Monitoring is a subset of observability.

---

# 4. Three Pillars of Observability

```
             Observability
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
  Metrics         Logs          Traces
```

Each pillar answers different questions.

---

# 5. Metrics

Metrics are numerical measurements collected over time.

Examples:

- CPU usage
- Memory usage
- Disk I/O
- Network traffic
- Request rate
- Error rate
- Request latency

Example:

```
CPU Usage

10%

↓

20%

↓

45%

↓

80%
```

Metrics are lightweight and ideal for dashboards and alerting.

---

## Common Kubernetes Metrics

Cluster:

- Node CPU
- Node memory
- Disk usage

Pods:

- CPU
- Memory
- Restarts

Applications:

- Requests per second
- Error rate
- Response time

---

# 6. Logs

Logs are timestamped records describing events.

Example:

```
2026-07-18T10:30:15Z INFO  Server started
2026-07-18T10:31:02Z ERROR Database connection failed
2026-07-18T10:31:10Z INFO  Retrying connection
```

Logs answer questions such as:

- What happened?
- When did it happen?
- Which component reported it?

---

## Kubernetes Log Sources

- Application containers
- kubelet
- API Server
- Scheduler
- Controller Manager
- etcd
- Ingress Controller

---

## View Logs

Current container:

```bash
kubectl logs pod-name
```

Follow logs:

```bash
kubectl logs -f pod-name
```

Previous container instance (useful after a restart):

```bash
kubectl logs --previous pod-name
```

---

# 7. Traces

Distributed tracing follows a request as it moves through multiple services.

Example:

```
User Request

↓

API Gateway

↓

Auth Service

↓

Order Service

↓

Payment Service

↓

Database
```

Each step contributes timing information.

---

## Why Tracing?

Imagine an application with:

- 30 microservices
- 200 API calls
- Thousands of requests per second

Tracing identifies:

- Where time is spent
- Which service is slow
- Where failures occur

---

# 8. Observability Architecture

```
Applications
       │
       ▼
Telemetry
       │
       ├────────────┬─────────────┐
       ▼            ▼             ▼
    Metrics       Logs         Traces
       │            │             │
       ▼            ▼             ▼
 Prometheus      Loki        Jaeger/Tempo
       │
       ▼
   Grafana
```

Grafana can visualize metrics, logs, and traces from multiple supported data sources.

---

# 9. Metrics Server

The Metrics Server provides **resource usage metrics** for Pods and Nodes.

Typical consumers include:

- `kubectl top`
- Horizontal Pod Autoscaler (HPA)

---

## Install (example)

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

---

## Verify

```bash
kubectl get deployment metrics-server -n kube-system
```

---

## View Node Metrics

```bash
kubectl top nodes
```

Example:

```
NAME      CPU    MEMORY
worker-1   25%     48%
worker-2   17%     39%
```

---

## View Pod Metrics

```bash
kubectl top pods -A
```

> **Note:** Metrics Server provides **current resource usage**. It is **not a long-term monitoring solution**.

---

# 10. Prometheus

Prometheus is the de facto standard for metrics collection in Kubernetes.

---

## Architecture

```
Targets

↓

Prometheus Server

↓

Time-Series Database

↓

PromQL

↓

Grafana
```

---

## Prometheus Pull Model

```
Node Exporter

↓

Prometheus

↓

Scrapes Metrics
```

Prometheus periodically pulls metrics from configured endpoints.

---

## Example Metrics Endpoint

```
/metrics
```

Applications expose metrics in a format that Prometheus can scrape.

---

# 11. kube-state-metrics

`kube-state-metrics` exposes information about Kubernetes objects.

Examples:

- Deployment replicas
- Pod status
- StatefulSet status
- Job completion
- DaemonSet availability

It reports **object state**, not CPU or memory utilization.

---

## Example Questions

- How many replicas are desired?
- How many are available?
- Which Pods are Pending?
- Which Jobs failed?

---

# 12. Node Exporter

Node Exporter collects operating system and hardware metrics from Kubernetes nodes.

Examples:

- CPU usage
- Memory usage
- Filesystem usage
- Disk I/O
- Network traffic

---

## Architecture

```
Linux Node

↓

Node Exporter

↓

Prometheus

↓

Grafana Dashboard
```

---

# Metrics Sources Comparison

| Component | Primary Data |
|-----------|--------------|
| Metrics Server | Current Pod & Node resource usage |
| Prometheus | Time-series metrics |
| kube-state-metrics | Kubernetes object state |
| Node Exporter | Operating system & hardware metrics |

---

# 13. Part 1 Summary

You learned:

- What observability is
- Monitoring vs observability
- The three pillars
- Metrics
- Logs
- Traces
- Metrics Server
- Prometheus
- kube-state-metrics
- Node Exporter

You now understand the foundational components of a Kubernetes observability platform.

# K8S-26 – Observability (Metrics, Logs & Traces) (Part 2)

> **Class Date:** 18-Jul-2026
>
> **Module:** Kubernetes Observability

---

# 📚 Table of Contents

14. Prometheus Architecture
15. PromQL Fundamentals
16. Grafana Dashboards
17. Alertmanager
18. Loki
19. Fluent Bit vs Fluentd
20. OpenTelemetry
21. Distributed Tracing
22. Jaeger & Grafana Tempo
23. Production Observability Architecture
24. Part 2 Summary

---

# 14. Prometheus Architecture

Prometheus periodically **scrapes** metrics from targets and stores them in its time-series database (TSDB).

```
          +------------------+
          |   Applications   |
          |   /metrics        |
          +---------+---------+
                    |
                    |
          +---------v---------+
          |   Node Exporter   |
          +---------+---------+
                    |
          +---------v---------+
          | kube-state-metrics|
          +---------+---------+
                    |
             (HTTP Scrape)
                    |
          +---------v---------+
          |    Prometheus     |
          |       TSDB        |
          +----+---------+----+
               |         |
          PromQL      Alert Rules
               |         |
               v         v
          Grafana   Alertmanager
```

Prometheus uses a **pull model**, periodically requesting metrics from configured endpoints.

---

## Important Components

| Component | Purpose |
|-----------|---------|
| Scrape Targets | Expose metrics |
| Prometheus Server | Collects and stores metrics |
| TSDB | Time-series database |
| PromQL | Query language |
| Alert Rules | Detect abnormal conditions |
| Alertmanager | Routes notifications |

---

# 15. PromQL Fundamentals

PromQL (Prometheus Query Language) retrieves and analyzes metrics.

---

## View a Metric

```
up
```

`up` indicates whether a target was successfully scraped.

---

## CPU Usage

```
node_cpu_seconds_total
```

---

## Container CPU Rate

```
rate(container_cpu_usage_seconds_total[5m])
```

Calculates the average CPU usage rate over the last five minutes.

---

## HTTP Request Rate

```
rate(http_requests_total[1m])
```

---

## Memory Usage

```
container_memory_working_set_bytes
```

---

## Filter by Label

```
up{job="kubernetes-pods"}
```

Labels make PromQL queries flexible and powerful.

---

# 16. Grafana Dashboards

Grafana visualizes data from multiple data sources.

```
Prometheus

↓

Grafana

↓

Dashboards
```

---

## Common Dashboard Panels

- CPU usage
- Memory usage
- Network traffic
- Disk utilization
- Pod restarts
- API latency
- Request rate
- Error rate

---

## Popular Dashboard Sources

Many teams start with community-maintained dashboards and customize them for their own environments.

---

# 17. Alertmanager

Prometheus evaluates alert rules.

When a rule is triggered:

```
Prometheus

↓

Alertmanager

↓

Notification

↓

Engineer
```

---

## Alert Flow

```
Metric

↓

Alert Rule

↓

Alertmanager

↓

Email / Slack / PagerDuty / Webhook
```

---

## Example Alert Rule

```yaml
groups:
- name: node-alerts

  rules:

  - alert: HighCPUUsage

    expr: node_cpu_seconds_total > 80

    for: 5m

    labels:
      severity: warning

    annotations:
      summary: High CPU usage detected
```

> **Note:** In real deployments, `node_cpu_seconds_total` is a cumulative counter. Production alerts typically use functions such as `rate()` and calculate CPU utilization percentages rather than comparing the raw counter value.

---

# 18. Loki

Loki is a log aggregation system designed to integrate closely with Grafana.

---

## Architecture

```
Pods

↓

Log Collector

↓

Loki

↓

Grafana
```

Unlike traditional log systems, Loki indexes metadata (labels) rather than the full log content.

---

## Benefits

- Lower storage overhead
- Kubernetes label integration
- Fast filtering by labels
- Native Grafana integration

---

# 19. Fluent Bit vs Fluentd

Both collect and forward logs.

| Feature | Fluent Bit | Fluentd |
|---------|------------|----------|
| Memory usage | Low | Higher |
| Performance | High | Moderate |
| Footprint | Lightweight | Larger |
| Plugins | Essential set | Very extensive |

---

## Recommendation

- **Fluent Bit**: Node-level log collection and forwarding.
- **Fluentd**: Complex log processing and transformations.

Many production environments use Fluent Bit on nodes because of its lower resource consumption.

---

# 20. OpenTelemetry

OpenTelemetry (OTel) is the CNCF standard for collecting telemetry.

It supports:

- Metrics
- Logs
- Traces

---

## Architecture

```
Application

↓

OTel SDK

↓

OTel Collector

↓

Backend
```

Backends may include systems for metrics, logs, or traces.

---

## Benefits

- Vendor-neutral instrumentation
- Standard telemetry format
- Automatic instrumentation for many languages
- Flexible export pipeline

---

# 21. Distributed Tracing

Tracing follows a request across services.

```
Client

↓

API Gateway

↓

Auth Service

↓

Order Service

↓

Payment Service

↓

Database
```

Each service creates one or more **spans**.

A collection of spans forms a **trace**.

---

## Trace Terminology

| Term | Meaning |
|------|---------|
| Trace | Entire request journey |
| Span | One operation within the trace |
| Parent Span | Caller operation |
| Child Span | Nested operation |

---

# 22. Jaeger & Grafana Tempo

Both are distributed tracing backends.

---

## Jaeger

Provides:

- Trace visualization
- Search
- Latency analysis
- Dependency graphs

---

## Grafana Tempo

Designed for scalable trace storage and integrates closely with Grafana.

---

## Comparison

| Feature | Jaeger | Tempo |
|----------|---------|--------|
| UI | Built-in | Uses Grafana |
| Storage | Multiple backends | Object storage focused |
| Ecosystem | Mature | Grafana ecosystem |

---

# 23. Production Observability Architecture

```
                    Applications
                          │
        ┌─────────────────┼──────────────────┐
        ▼                 ▼                  ▼
     Metrics            Logs              Traces
        │                 │                  │
        ▼                 ▼                  ▼
 Prometheus         Fluent Bit         OpenTelemetry
        │                 │                  │
        ▼                 ▼                  ▼
   Alertmanager         Loki         Jaeger / Tempo
        │                 │                  │
        └──────────────┬──┴──────────────────┘
                       ▼
                    Grafana
```

Grafana provides a unified interface for dashboards and, depending on configuration, can correlate metrics, logs, and traces.

---

# Observability Workflow

```
User reports slow response

↓

Dashboard shows latency increase

↓

Inspect logs

↓

Open trace

↓

Identify slow service

↓

Resolve issue
```

This workflow illustrates how the three pillars complement one another during troubleshooting.

---

# 24. Part 2 Summary

You learned:

- Prometheus architecture
- PromQL basics
- Grafana dashboards
- Alertmanager
- Loki
- Fluent Bit vs Fluentd
- OpenTelemetry
- Distributed tracing
- Jaeger
- Grafana Tempo
- Production observability architecture

You now understand how telemetry is collected, stored, visualized, and correlated in a modern Kubernetes platform.

# K8S-26 – Observability (Metrics, Logs & Traces) (Part 3)

> **Class Date:** 18-Jul-2026
>
> **Module:** Kubernetes Observability

---

# 📚 Table of Contents

25. Installing the Monitoring Stack
26. ServiceMonitor & PodMonitor
27. Recording Rules
28. SLI, SLO & SLA
29. Golden Signals
30. RED & USE Methodologies
31. Troubleshooting Workflow
32. Production Best Practices
33. Hands-on Labs
34. CKA / CKAD / CKS Interview Questions
35. Observability Cheat Sheet
36. Real Production Case Studies
37. Chapter Summary

---

# 25. Installing the Monitoring Stack

A common production deployment uses the **kube-prometheus-stack** Helm chart.

```
Helm

↓

kube-prometheus-stack

↓

Prometheus
Grafana
Alertmanager
Node Exporter
kube-state-metrics
```

Install:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

Verify:

```bash
kubectl get pods -n monitoring
```

Expected components:

- Prometheus
- Grafana
- Alertmanager
- Node Exporter
- kube-state-metrics
- Prometheus Operator

---

# 26. ServiceMonitor & PodMonitor

The Prometheus Operator introduces **Custom Resources** that define how Prometheus discovers scrape targets.

## ServiceMonitor

Monitors metrics exposed through a Kubernetes **Service**.

```
Application

↓

Service

↓

ServiceMonitor

↓

Prometheus
```

Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor

metadata:
  name: payment-api

spec:
  selector:
    matchLabels:
      app: payment-api

  endpoints:
  - port: metrics
    interval: 30s
```

---

## PodMonitor

Scrapes Pods directly without requiring a Service.

```
Pods

↓

PodMonitor

↓

Prometheus
```

Useful for workloads that do not expose a stable Service.

---

# 27. Recording Rules

Complex PromQL queries can be expensive.

Recording rules precompute frequently used queries and store the results as new time series.

Example:

```yaml
groups:
- name: application-rules

  rules:
  - record: job:http_requests:rate5m

    expr: rate(http_requests_total[5m])
```

Benefits:

- Faster dashboards
- Simpler alert rules
- Reduced query load

---

# 28. SLI, SLO & SLA

## SLI (Service Level Indicator)

A measurable characteristic.

Examples:

- Request success rate
- Latency
- Availability

---

## SLO (Service Level Objective)

The target value for an SLI.

Example:

```
99.9% availability
```

---

## SLA (Service Level Agreement)

A business agreement between provider and customer.

Example:

```
99.9% uptime

↓

Financial penalties if violated
```

---

## Relationship

```
Metric

↓

SLI

↓

SLO

↓

SLA
```

---

# Error Budget

```
SLO = 99.9%

↓

Allowed downtime ≈ 0.1%
```

The error budget helps teams balance reliability improvements with feature delivery.

---

# 29. Golden Signals

Google SRE identifies four Golden Signals.

```
Latency

Traffic

Errors

Saturation
```

---

## Latency

How long requests take.

---

## Traffic

System demand.

Examples:

- Requests/second
- Connections
- Transactions

---

## Errors

Examples:

- HTTP 500 responses
- Failed jobs
- Failed database queries

---

## Saturation

Resource utilization approaching system limits.

Examples:

- CPU
- Memory
- Queue length

---

# 30. RED & USE Methodologies

## RED

Designed for services.

```
Rate

Errors

Duration
```

Monitor:

- Request rate
- Error rate
- Response duration

---

## USE

Designed for infrastructure.

```
Utilization

Saturation

Errors
```

Monitor:

- CPU utilization
- Queue saturation
- Hardware or infrastructure errors

---

## When to Use

| Method | Best For |
|---------|----------|
| RED | Applications & APIs |
| USE | Infrastructure |

Many production environments use both together.

---

# 31. Troubleshooting Workflow

Example:

```
Alert

↓

Open Grafana Dashboard

↓

High CPU

↓

Check Pod Logs

↓

Review Trace

↓

Identify Root Cause

↓

Deploy Fix
```

---

## Useful Commands

View logs:

```bash
kubectl logs <pod>
```

Describe Pod:

```bash
kubectl describe pod <pod>
```

Events:

```bash
kubectl get events --sort-by=.lastTimestamp
```

Node metrics:

```bash
kubectl top nodes
```

Pod metrics:

```bash
kubectl top pods -A
```

---

# 32. Production Best Practices

---

## Use Labels Consistently

Metrics should include meaningful labels such as:

- application
- namespace
- environment
- version

Avoid high-cardinality labels (for example, unique request IDs or user IDs) because they can significantly increase storage and query costs.

---

## Centralize Logs

Avoid relying on node-local logs.

Forward logs to a centralized platform.

---

## Retention Policies

Define different retention periods based on operational and compliance needs.

Example:

- Metrics: 30–90 days
- Logs: 7–30 days (or longer if required)
- Traces: Shorter retention due to storage cost

Actual values depend on your organization's requirements.

---

## Secure Observability

Protect:

- Dashboards
- Metrics endpoints
- Logs
- Tracing backends

Use:

- RBAC
- Authentication
- TLS
- NetworkPolicies

---

## Alert Quality

Alerts should be:

- Actionable
- Meaningful
- Low noise

Avoid excessive alerting that leads to alert fatigue.

---

# 33. Hands-on Labs

## Lab 1

Install the monitoring stack.

Verify all Pods are healthy.

---

## Lab 2

Create a ServiceMonitor.

Confirm the target appears in Prometheus.

---

## Lab 3

Build a Grafana dashboard showing:

- CPU
- Memory
- Pod restarts

---

## Lab 4

Create a Prometheus alert for high CPU utilization using an appropriate PromQL expression.

---

## Lab 5

Generate application traffic and:

- Observe metrics
- Inspect logs
- View traces (if tracing is configured)

Correlate the data to identify a simulated issue.

---

# 34. CKA / CKAD / CKS Interview Questions

> **Note:** Deep Prometheus and Grafana administration is outside the current CKA objectives, but observability concepts are highly valuable in CKAD, CKS, SRE, DevOps, and production Kubernetes interviews.

## Beginner

1. What is observability?
2. What are the three pillars?
3. What is Prometheus?
4. What is Grafana?
5. What is Metrics Server?

---

## Intermediate

1. Explain ServiceMonitor.
2. Explain PodMonitor.
3. What is Alertmanager?
4. Compare Metrics Server and Prometheus.
5. What is OpenTelemetry?

---

## Advanced

1. Design a production observability platform.
2. Explain RED vs USE.
3. Explain SLI, SLO, and SLA.
4. How would you troubleshoot latency spikes?
5. How would you reduce Prometheus storage costs?

---

## Scenario-Based

### Scenario 1

CPU usage is high, but application latency is normal.

Would you scale immediately? What else would you investigate?

---

### Scenario 2

A Grafana dashboard shows Pod restarts increasing.

Which Kubernetes resources and telemetry would you inspect next?

---

### Scenario 3

Prometheus storage usage grows rapidly.

What factors might contribute to this?

---

### Scenario 4

A request is slow only in production.

How could metrics, logs, and traces help isolate the issue?

---

### Scenario 5

Your alerts are firing too frequently.

How would you improve alert quality?

---

# 35. Observability Cheat Sheet

Node metrics:

```bash
kubectl top nodes
```

Pod metrics:

```bash
kubectl top pods -A
```

Logs:

```bash
kubectl logs <pod>
kubectl logs -f <pod>
kubectl logs --previous <pod>
```

Events:

```bash
kubectl get events --sort-by=.lastTimestamp
```

Install monitoring stack:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack
```

Prometheus targets:

```
Status → Targets
```

Grafana:

```
Dashboards → Explore
```

---

# 36. Real Production Case Studies

## Microservices Platform

```
Applications

↓

Prometheus

↓

Grafana

↓

Alertmanager

↓

On-call Engineer
```

Alerts notify engineers of service degradation.

---

## Logging Pipeline

```
Pods

↓

Fluent Bit

↓

Loki

↓

Grafana
```

Logs from all nodes are centralized and searchable.

---

## Distributed Tracing

```
Client

↓

API Gateway

↓

Order Service

↓

Payment Service

↓

Database
```

Tracing reveals where request latency is introduced.

---

## Full Observability Platform

```
Applications
       │
       ├───────────┬─────────────┐
       ▼           ▼             ▼
   Metrics       Logs         Traces
       │           │             │
       ▼           ▼             ▼
 Prometheus     Loki     OpenTelemetry
       │           │             │
       └──────┬────┴──────┬──────┘
              ▼           ▼
          Alertmanager  Tempo/Jaeger
                  │
                  ▼
               Grafana
```

---

# 37. Chapter Summary

Congratulations! 🎉

You have completed **K8S-26 – Observability (Metrics, Logs & Traces)**.

In this chapter, you learned:

- Observability fundamentals
- Metrics, logs, and traces
- Metrics Server
- Prometheus
- PromQL basics
- Grafana
- Alertmanager
- Loki
- Fluent Bit vs Fluentd
- OpenTelemetry
- Jaeger and Tempo
- ServiceMonitor & PodMonitor
- Recording rules
- SLI, SLO, SLA
- Golden Signals
- RED & USE
- Troubleshooting workflows
- Production best practices
- Hands-on labs
- Interview questions

You now have a strong foundation for designing, operating, and troubleshooting a production-grade Kubernetes observability platform.

