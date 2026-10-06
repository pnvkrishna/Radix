# K8S-30 – Service Mesh (Istio & Linkerd) (Part 1)

> **Class Date:** 22-Jul-2026
>
> **Module:** Service Mesh

---

# 📚 Table of Contents

1. Introduction
2. What is a Service Mesh?
3. Why Do We Need a Service Mesh?
4. Problems Without a Service Mesh
5. Service Mesh Architecture
6. Data Plane vs Control Plane
7. Sidecar Proxy Concept
8. Envoy Proxy
9. Istio Overview
10. Linkerd Overview
11. Part 1 Summary

---

# 1. Introduction

As Kubernetes deployments grow from a few services to dozens or hundreds of microservices, communication between services becomes increasingly complex.

Typical concerns include:

- Encryption
- Authentication
- Authorization
- Traffic routing
- Retries
- Timeouts
- Load balancing
- Observability

Embedding all of this logic into every application leads to duplicated code and inconsistent behavior.

A **Service Mesh** moves these cross-cutting concerns into the platform.

---

# 2. What is a Service Mesh?

A Service Mesh is a dedicated infrastructure layer that manages communication between services.

Instead of:

```
Service A

↓

Service B
```

Communication flows through proxies managed by the mesh.

```
Service A

↓

Proxy

↓

Proxy

↓

Service B
```

Applications focus on business logic.

The mesh handles networking concerns.

---

# 3. Why Do We Need a Service Mesh?

Without a mesh, every application team often implements:

- TLS
- Retries
- Timeouts
- Circuit breakers
- Metrics
- Logging
- Authentication

This creates inconsistency.

With a Service Mesh:

```
Applications

↓

Service Mesh

↓

Networking Features
```

Features become standardized across services.

---

# 4. Problems Without a Service Mesh

Example:

```
Order Service

↓

Payment Service

↓

Inventory Service

↓

Notification Service
```

Questions arise:

- How do we encrypt traffic?
- How do we retry failed requests?
- How do we split traffic safely?
- How do we collect traces?
- How do we authenticate services?

Without a mesh, each application may solve these problems differently.

---

# 5. Service Mesh Architecture

```
              Control Plane
                     │
                     ▼
          Configuration & Policies
                     │
      ┌──────────────┼──────────────┐
      ▼              ▼              ▼
  Envoy Proxy    Envoy Proxy    Envoy Proxy
      │              │              │
      ▼              ▼              ▼
 Service A      Service B      Service C
```

The control plane distributes configuration.

The data plane enforces it.

---

# 6. Data Plane vs Control Plane

| Control Plane | Data Plane |
|---------------|------------|
| Manages configuration | Handles application traffic |
| Distributes policies | Enforces policies |
| Controls routing | Processes requests |
| No application traffic | Carries application traffic |

---

## Flow

```
Control Plane

↓

Configure Proxies

↓

Proxy Handles Traffic
```

---

# 7. Sidecar Proxy Concept

A sidecar is an additional container running in the same Pod as the application.

```
Pod

├── Application

└── Proxy
```

All inbound and outbound traffic passes through the proxy.

---

## Request Flow

```
Application

↓

Sidecar Proxy

↓

Network

↓

Sidecar Proxy

↓

Application
```

The application is generally unaware of the proxy.

---

# 8. Envoy Proxy

Envoy is a high-performance Layer 7 proxy used by Istio and several other service mesh implementations.

Capabilities include:

- Load balancing
- TLS termination
- Retries
- Timeouts
- Circuit breaking
- Metrics
- Distributed tracing
- Access logging

---

## Traffic Flow

```
Application

↓

Envoy

↓

Destination Envoy

↓

Destination Application
```

---

# 9. Istio Overview

Istio is one of the most feature-rich service meshes.

Key capabilities:

- Traffic management
- Mutual TLS (mTLS)
- Authorization policies
- Observability
- Traffic splitting
- Fault injection
- Multi-cluster support
- Ambient Mesh support (newer architecture)

---

## Typical Architecture

```
               Istiod
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
    Envoy       Envoy       Envoy
      │           │           │
 Service A   Service B   Service C
```

`istiod` manages configuration and certificate distribution.

---

# 10. Linkerd Overview

Linkerd is a lightweight CNCF service mesh.

Characteristics:

- Simpler architecture
- Lower operational overhead
- Fast installation
- Automatic mTLS
- Strong focus on reliability and simplicity

---

## Linkerd Architecture

```
Control Plane

↓

Proxy

↓

Application
```

Compared to Istio, Linkerd emphasizes ease of operation over extensive feature sets.

---

# Istio vs Linkerd (High-Level)

| Feature | Istio | Linkerd |
|---------|--------|----------|
| Feature Rich | ✅ | Moderate |
| Operational Simplicity | Moderate | ✅ |
| Built-in UI | Via integrations (e.g., Kiali) | Via extensions |
| Traffic Management | Advanced | Good |
| Ambient Mesh | ✅ | ❌ |
| Learning Curve | Higher | Lower |

---

# When Should You Use a Service Mesh?

Consider adopting one when:

- Running many microservices
- Requiring service-to-service encryption
- Needing advanced traffic routing
- Implementing progressive delivery
- Requiring consistent observability
- Enforcing zero-trust networking

For small clusters with only a few services, a service mesh may introduce unnecessary operational complexity.

---

# 11. Part 1 Summary

You learned:

- What a Service Mesh is
- Why it exists
- Problems it solves
- Service Mesh architecture
- Control Plane vs Data Plane
- Sidecar proxies
- Envoy
- Istio overview
- Linkerd overview
- When to use a Service Mesh

You now understand the architectural foundations of modern Kubernetes service meshes.

# K8S-30 – Service Mesh (Istio & Linkerd) (Part 2)

> **Class Date:** 22-Jul-2026
>
> **Module:** Istio Traffic Management

---

# 📚 Table of Contents

12. Installing Istio
13. istiod
14. Sidecar Injection
15. Ambient Mesh
16. VirtualService
17. DestinationRule
18. Gateway
19. ServiceEntry
20. Traffic Management
21. Part 2 Summary

---

# 12. Installing Istio

Install the Istio CLI (`istioctl`) and then install Istio into the cluster.

Example:

```bash
istioctl install
```

For production environments, review the installation profile and customize settings rather than relying solely on defaults.

---

## Verify Installation

```bash
kubectl get pods -n istio-system
```

Typical components include:

- istiod
- ingress gateway (if installed)
- additional components depending on the chosen profile

---

# 13. istiod

`istiod` is the primary control plane component in modern Istio.

Responsibilities:

- Configuration distribution
- Service discovery
- Certificate management
- Proxy configuration
- Policy distribution

---

## Architecture

```
            istiod
               │
      ┌────────┼────────┐
      ▼        ▼        ▼
   Envoy     Envoy     Envoy
      │        │        │
 Service A  Service B  Service C
```

The proxies receive configuration dynamically from `istiod`.

---

# 14. Sidecar Injection

A sidecar proxy is added automatically to application Pods.

Before:

```
Pod

└── Application
```

After injection:

```
Pod

├── Application

└── Envoy Proxy
```

---

## Enable Automatic Injection

Label a namespace:

```bash
kubectl label namespace default istio-injection=enabled
```

New Pods created in that namespace will receive sidecars.

> Restart or recreate existing Pods after enabling injection so the sidecar can be added.

---

# 15. Ambient Mesh

Ambient Mesh reduces the operational cost of traditional sidecars.

Instead of placing a proxy inside every Pod:

```
Application

↓

ztunnel

↓

Network
```

---

## Components

### ztunnel

Handles:

- Secure communication
- Identity
- Mutual TLS
- Basic Layer 4 traffic processing

---

### Waypoint Proxy

Provides Layer 7 features such as:

- HTTP routing
- Advanced traffic policies
- Authorization
- Observability enhancements

Only workloads requiring these capabilities need waypoint proxies.

---

## Traditional vs Ambient

Traditional:

```
Pod

├── App

└── Envoy
```

Ambient:

```
App

↓

ztunnel

↓

Waypoint (optional)

↓

Destination
```

---

# 16. VirtualService

A `VirtualService` defines **how traffic should be routed**.

Example:

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService

metadata:
  name: payment

spec:
  hosts:
  - payment

  http:
  - route:
    - destination:
        host: payment
```

---

## Key Fields

| Field | Purpose |
|--------|----------|
| hosts | Target service(s) |
| http | HTTP routing rules |
| route | Destination rules |
| match | Traffic matching |
| rewrite | URL rewriting |
| timeout | Request timeout |
| retries | Retry policy |

---

# 17. DestinationRule

A `DestinationRule` defines policies applied **after** traffic reaches the destination service.

Example:

```yaml
apiVersion: networking.istio.io/v1
kind: DestinationRule

metadata:
  name: payment

spec:
  host: payment

  trafficPolicy:
    loadBalancer:
      simple: ROUND_ROBIN
```

---

## Common Policies

- Load balancing
- Connection pools
- Circuit breakers
- Outlier detection
- TLS settings

---

# 18. Gateway

A `Gateway` controls external traffic entering or leaving the mesh.

```
Internet

↓

Gateway

↓

VirtualService

↓

Application
```

---

Example:

```yaml
apiVersion: networking.istio.io/v1
kind: Gateway

metadata:
  name: web-gateway
```

Typical configuration includes:

- Hosts
- Ports
- TLS
- Selectors

---

# 19. ServiceEntry

By default, Istio primarily manages traffic for services it knows about.

`ServiceEntry` allows external services to be represented within the mesh.

Example:

```
Application

↓

ServiceEntry

↓

api.example.com
```

Common use cases:

- External APIs
- SaaS integrations
- Legacy services

---

# 20. Traffic Management

One of Istio's strongest capabilities is advanced traffic routing.

---

## Traffic Splitting

Example:

```
Version 1

90%

↓

Users

↓

Version 2

10%
```

Typical configuration combines:

- `VirtualService`
- `DestinationRule`

---

## Canary Deployment

```
5%

↓

10%

↓

25%

↓

50%

↓

100%
```

Increase traffic only after validating health and performance.

---

## Blue–Green Deployment

```
Blue Environment

↓

Green Environment

↓

Traffic Switch
```

Rollback simply directs traffic back to the previous environment.

---

## Retry Policy

Example:

```yaml
retries:
  attempts: 3
```

Retries help mitigate transient failures.

---

## Timeout

Example:

```yaml
timeout: 5s
```

Timeouts prevent requests from waiting indefinitely.

---

## Fault Injection

Used primarily for testing.

Examples:

- Artificial delay
- Injected HTTP errors

This supports chaos engineering and resilience testing.

---

# Traffic Management Flow

```
Client

↓

Gateway

↓

VirtualService

↓

DestinationRule

↓

Application
```

---

# Production Best Practices

### Keep Routing Simple

Avoid overly complex routing rules unless there is a clear operational need.

---

### Gradual Rollouts

Prefer:

- Canary
- Progressive delivery

Avoid switching all production traffic at once when introducing major changes.

---

### Observe Before Expanding

Monitor:

- Latency
- Error rate
- Resource usage
- User impact

before increasing traffic to a new version.

---

### Secure Communication

Enable mutual TLS where appropriate to encrypt service-to-service communication and verify workload identities.

---

# 21. Part 2 Summary

You learned:

- Installing Istio
- `istiod`
- Sidecar injection
- Ambient Mesh
- `VirtualService`
- `DestinationRule`
- `Gateway`
- `ServiceEntry`
- Traffic routing
- Traffic splitting
- Canary deployments
- Blue–Green deployments
- Retry policies
- Timeouts
- Fault injection

You now understand how Istio manages traffic in production Kubernetes environments.

# K8S-30 – Service Mesh (Istio & Linkerd) (Part 3)

> **Class Date:** 22-Jul-2026
>
> **Module:** Service Mesh Security & Observability

---

# 📚 Table of Contents

22. Mutual TLS (mTLS)
23. PeerAuthentication
24. AuthorizationPolicy
25. RequestAuthentication (JWT)
26. Observability with Kiali, Prometheus, Grafana & Jaeger
27. Linkerd Operations
28. Multi-Cluster Service Mesh
29. Production Best Practices
30. Hands-on Labs
31. CKA / CKAD / CKS Interview Questions
32. Service Mesh Cheat Sheet
33. Chapter Summary

---

# 22. Mutual TLS (mTLS)

Mutual TLS encrypts traffic **and** verifies the identities of both communicating workloads.

```
Service A
    │
    │  Identity + Encryption
    ▼
Service B
```

Unlike standard TLS, where only the server is authenticated, **mTLS authenticates both client and server**.

---

## Benefits

- Encryption in transit
- Strong workload identity
- Protection against impersonation
- Zero-trust networking foundation

---

## Modes

| Mode | Description |
|------|-------------|
| STRICT | Only mTLS traffic allowed |
| PERMISSIVE | Accepts both plaintext and mTLS |
| DISABLE | mTLS disabled |

Production environments generally aim for **STRICT** after validating application compatibility.

---

# 23. PeerAuthentication

`PeerAuthentication` controls how workloads accept inbound connections.

Example:

```yaml
apiVersion: security.istio.io/v1
kind: PeerAuthentication

metadata:
  name: default

spec:
  mtls:
    mode: STRICT
```

---

## Scope

Policies may apply at:

- Mesh level
- Namespace level
- Workload level

---

## Migration Strategy

```
DISABLE

↓

PERMISSIVE

↓

STRICT
```

Move gradually to avoid disrupting existing workloads.

---

# 24. AuthorizationPolicy

Authentication answers:

> **Who are you?**

Authorization answers:

> **What are you allowed to do?**

---

Example:

```yaml
apiVersion: security.istio.io/v1
kind: AuthorizationPolicy

metadata:
  name: payment-policy
```

Typical rules include:

- Allowed principals
- Namespaces
- HTTP methods
- Paths
- Ports

---

## Example Flow

```
Client

↓

Authenticated

↓

Authorization Check

↓

Allow / Deny

↓

Application
```

---

# 25. RequestAuthentication (JWT)

Istio can validate JSON Web Tokens (JWTs) before requests reach an application.

```
Client

↓

JWT

↓

Istio Validation

↓

Application
```

---

Typical configuration defines:

- Issuer
- JWKS URI
- Token validation rules

After successful validation, an `AuthorizationPolicy` can make decisions based on JWT claims.

---

# 26. Observability with Kiali, Prometheus, Grafana & Jaeger

A service mesh generates rich telemetry.

---

## Kiali

Provides a topology view of services.

```
Frontend

↓

Orders

↓

Payments

↓

Database
```

Useful for:

- Traffic visualization
- Configuration inspection
- Health status

---

## Prometheus

Collects:

- Request rate
- Error rate
- Latency
- Traffic volume

---

## Grafana

Visualizes metrics through dashboards.

Common dashboards include:

- Service latency
- Request throughput
- Error percentage
- mTLS status (if available through dashboards)

---

## Jaeger

Displays distributed traces.

```
Client

↓

Gateway

↓

Orders

↓

Payments

↓

Database
```

Traces help identify where latency is introduced.

---

# Observability Flow

```
Application

↓

Envoy Proxy

↓

Telemetry

↓

Prometheus

↓

Grafana

↓

Operators
```

Tracing data can also be exported to Jaeger or Tempo.

---

# 27. Linkerd Operations

Install Linkerd:

```bash
linkerd install | kubectl apply -f -
```

Verify:

```bash
linkerd check
```

Inject proxies:

```bash
linkerd inject deployment.yaml | kubectl apply -f -
```

---

## Common Operations

```bash
linkerd stat deploy

linkerd top deploy
```

These commands provide traffic and workload statistics.

---

# 28. Multi-Cluster Service Mesh

```
            Global DNS
                 │
      ┌──────────┴──────────┐
      ▼                     ▼
 Cluster A             Cluster B
      │                     │
      └──────────┬──────────┘
                 ▼
          Secure Mesh Communication
```

Benefits:

- Regional resilience
- Workload mobility
- Shared service discovery
- Secure cross-cluster communication

---

# 29. Production Best Practices

## Enable mTLS

Encrypt service-to-service traffic wherever practical.

---

## Principle of Least Privilege

Restrict communication using:

- AuthorizationPolicy
- Kubernetes NetworkPolicies (where applicable)

---

## Monitor Telemetry

Observe:

- Latency
- Error rates
- Retries
- Timeouts
- Connection failures

---

## Avoid Over-Engineering

Not every workload requires:

- Complex routing
- Fault injection
- Advanced traffic policies

Use advanced features only where they provide operational value.

---

## Progressive Delivery

Combine:

- GitOps
- Service Mesh
- Progressive rollout tools

for safer deployments.

---

# Production Architecture

```
                 Internet
                     │
                     ▼
            Istio Ingress Gateway
                     │
                     ▼
              VirtualService
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
    Orders      Payments     Inventory
        │            │            │
        ▼            ▼            ▼
     Envoy        Envoy        Envoy
        │            │            │
        └────────────┼────────────┘
                     ▼
               Prometheus
                     │
      ┌──────────────┴──────────────┐
      ▼                             ▼
   Grafana                       Jaeger
                     │
                     ▼
                   Kiali
```

---

# 30. Hands-on Labs

## Lab 1

Install Istio in a test cluster.

Verify:

- `istiod`
- Ingress gateway
- Sidecar injection

---

## Lab 2

Deploy two services.

Enable automatic sidecar injection.

Confirm that communication flows through the mesh.

---

## Lab 3

Create:

- `VirtualService`
- `DestinationRule`

Perform a canary deployment with a small percentage of traffic.

---

## Lab 4

Enable **STRICT** mTLS for a namespace.

Verify encrypted service-to-service communication.

---

## Lab 5

Install:

- Prometheus
- Grafana
- Kiali

Observe service topology, request rates, and traces (if tracing is configured).

---

# 31. CKA / CKAD / CKS Interview Questions

> **Note:** Service mesh technologies are not primary CKA/CKAD/CKS exam objectives, but they are frequently discussed in senior Kubernetes, SRE, Platform Engineering, and cloud-native architecture interviews.

## Beginner

1. What is a Service Mesh?
2. Why use Envoy?
3. What is mTLS?
4. What is a sidecar proxy?
5. Compare Istio and Linkerd.

---

## Intermediate

1. Explain `VirtualService`.
2. Explain `DestinationRule`.
3. What is `PeerAuthentication`?
4. What is `AuthorizationPolicy`?
5. What is Ambient Mesh?

---

## Advanced

1. Design a secure service mesh for a production platform.
2. Explain zero-trust networking with Istio.
3. How would you troubleshoot mTLS failures?
4. Compare sidecar mode and Ambient Mesh.
5. Design a multi-cluster service mesh architecture.

---

## Scenario-Based

### Scenario 1

A service cannot communicate with another service after enabling **STRICT** mTLS.

What would you verify first?

---

### Scenario 2

Only 10% of users should receive a new application version.

How would you configure traffic management?

---

### Scenario 3

A request is slow, but CPU usage is low.

Which observability tools would help isolate the latency?

---

### Scenario 4

A JWT-authenticated request is unexpectedly rejected.

Which authentication and authorization resources would you inspect?

---

### Scenario 5

A team wants secure service-to-service communication but prefers lower operational overhead than sidecar mode.

How might Ambient Mesh influence your recommendation?

---

# 32. Service Mesh Cheat Sheet

## Install Istio

```bash
istioctl install
```

---

## Enable Sidecar Injection

```bash
kubectl label namespace default istio-injection=enabled
```

---

## Verify Pods

```bash
kubectl get pods -n istio-system
```

---

## Linkerd

```bash
linkerd check

linkerd stat deploy

linkerd top deploy
```

---

## Common Istio Resources

- VirtualService
- DestinationRule
- Gateway
- ServiceEntry
- PeerAuthentication
- AuthorizationPolicy
- RequestAuthentication

---

# 33. Chapter Summary

Congratulations! 🎉

You have completed **K8S-30 – Service Mesh (Istio & Linkerd)**.

In this chapter, you learned:

- Service Mesh fundamentals
- Data plane vs control plane
- Envoy
- Istio
- Linkerd
- Ambient Mesh
- Sidecar injection
- `VirtualService`
- `DestinationRule`
- `Gateway`
- `ServiceEntry`
- mTLS
- `PeerAuthentication`
- `AuthorizationPolicy`
- `RequestAuthentication`
- Observability integration
- Multi-cluster service mesh
- Production best practices
- Hands-on labs
- Interview questions

You now have a production-ready understanding of how service meshes provide **secure, observable, and intelligent communication** for Kubernetes-based microservices.


