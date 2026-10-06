# Kubernetes Class 07 - Services (Part 1)

> **Class Date:** 29-Jun-2026
>
> **Module:** Kubernetes Services


---

# 📚 Table of Contents

1. Introduction
2. Why Services?
3. The Problem with Pods
4. What is a Service?
5. Service Architecture
6. Service Discovery
7. Labels & Selectors
8. ClusterIP
9. Endpoints & EndpointSlices
10. kube-proxy
11. Summary

---

# 1. Introduction

In the previous chapter, we learned that Deployments create and manage Pods.

However, Pods are **ephemeral**.

This means they can:

- Be deleted
- Be recreated
- Receive a new IP address
- Move to another node

If clients communicate directly with Pod IP addresses, the application becomes unreliable.

This is why Kubernetes provides **Services**.

A Service gives applications a **stable network identity**, allowing clients to communicate without needing to know which Pods are currently running.

---

# 2. Why Services?

Imagine an application with three Pods.

```
Deployment

↓

Pod-1

10.244.0.10

Pod-2

10.244.0.11

Pod-3

10.244.0.12
```

A client connects directly to:

```
10.244.0.10
```

Now Pod-1 crashes.

ReplicaSet creates a replacement.

```
Old Pod

10.244.0.10

↓

Deleted

↓

New Pod

10.244.1.25
```

The client's saved IP address is now invalid.

Connections fail.

---

# Real World Example

Think of a customer support team.

Instead of memorizing every employee's mobile number,

customers call one official support number.

```
Customer

↓

Support Number

↓

Available Agent
```

The support number never changes.

Employees may join or leave,

but customers continue using the same number.

A Kubernetes Service works in exactly the same way.

---

# 3. The Problem with Pods

Pods are designed to be temporary.

They can disappear because of:

- Node failure
- Application crash
- Rolling update
- Scaling
- Manual deletion

Example:

```
Client

↓

Pod

↓

Deleted
```

Communication immediately breaks.

Production systems require a permanent endpoint.

---

# 4. What is a Service?

A Service is a Kubernetes resource that provides a stable virtual IP address and DNS name for a group of Pods.

A Service:

- Selects Pods using labels
- Provides a stable IP
- Balances traffic
- Hides Pod IP changes

Instead of talking to individual Pods,

applications communicate with the Service.

---

# Service Architecture

```
                Client

                  │

                  ▼

            Kubernetes Service

                  │

        ┌─────────┼─────────┐

        ▼         ▼         ▼

      Pod-1     Pod-2     Pod-3

       Ready     Ready     Ready
```

The client never communicates directly with the Pods.

---

# 5. How Services Work

Workflow

```
Client

↓

Service IP

↓

kube-proxy

↓

Matching Pod

↓

Response
```

Notice that the client never knows which Pod handled the request.

The Service selects an available Pod automatically.

---

# 6. Service Discovery

Every Service receives:

- A stable Cluster IP
- A DNS name

Instead of using IP addresses,

applications usually communicate using DNS.

Example:

```
database.default.svc.cluster.local
```

or simply

```
database
```

when the client is in the same namespace.

This makes applications portable and resilient.

---

# DNS Resolution

```
Application

↓

DNS Query

↓

CoreDNS

↓

Service IP

↓

Backend Pod
```

CoreDNS is responsible for translating Service names into IP addresses.

---

# 7. Labels & Selectors

A Service does not know Pod names.

Instead, it searches for Pods using labels.

Example Pod:

```yaml
labels:
  app: nginx
```

Service selector:

```yaml
selector:
  app: nginx
```

Relationship

```
Service

↓

Selector

↓

app=nginx

↓

Matching Pods
```

If labels do not match,

the Service has no backend Pods.

---

# Verify Labels

```bash
kubectl get pods --show-labels
```

---

# Verify Service

```bash
kubectl get svc
```

---

# 8. ClusterIP

ClusterIP is the default Service type.

It exposes an application **inside the cluster only**.

```
Application A

↓

ClusterIP Service

↓

Pods
```

External users cannot access it directly.

---

## Default Service Type

If no type is specified:

```yaml
spec:
  type: ClusterIP
```

Kubernetes automatically creates a ClusterIP Service.

---

# ClusterIP Architecture

```
+------------------------------------------+

Cluster

Client

↓

ClusterIP

↓

Pod

Pod

Pod

+------------------------------------------+
```

This is the most commonly used Service type.

Examples:

- Backend APIs
- Databases
- Internal microservices
- Redis
- RabbitMQ

---

# Example YAML

```yaml
apiVersion: v1

kind: Service

metadata:
  name: nginx-service

spec:

  selector:
    app: nginx

  ports:

  - port: 80

    targetPort: 80

  type: ClusterIP
```

---

# YAML Explanation

## selector

Finds matching Pods.

```yaml
selector:
  app: nginx
```

---

## port

The Service port.

```yaml
port: 80
```

Clients connect to this port.

---

## targetPort

The container port inside the Pod.

```yaml
targetPort: 80
```

Traffic is forwarded here.

---

## type

```yaml
ClusterIP
```

Internal-only communication.

---

# Create Service

```bash
kubectl apply -f service.yaml
```

---

# Verify Service

```bash
kubectl get svc
```

Example:

```
NAME            TYPE        CLUSTER-IP
nginx-service   ClusterIP   10.96.145.20
```

---

# 9. Endpoints & EndpointSlices

A Service forwards traffic to backend Pods.

But how does it know where the Pods are?

Kubernetes automatically creates **Endpoints** (and, in modern Kubernetes, **EndpointSlices**) for matching Pods.

```
Service

↓

EndpointSlice

↓

Pod-1

Pod-2

Pod-3
```

Whenever Pods are added or removed,

EndpointSlices are updated automatically.

---

## View EndpointSlices

```bash
kubectl get endpointslices
```

---

## View Endpoints

```bash
kubectl get endpoints
```

---

# 10. kube-proxy

Every Kubernetes node runs **kube-proxy**.

Its responsibility is to implement Service networking.

Workflow

```
Client

↓

Service IP

↓

kube-proxy

↓

Pod Selected

↓

Traffic Forwarded
```

kube-proxy watches the Kubernetes API and updates node networking rules whenever Services or Pods change.

---

## Operating Modes

Historically, kube-proxy has supported multiple modes:

- **iptables** (most common on Linux)
- **IPVS** (optimized for very large numbers of Services)
- **Userspace** (legacy; rarely used today)

The exact mode depends on cluster configuration and Kubernetes version.

---

# Verify kube-proxy

```bash
kubectl get pods -n kube-system
```

Look for:

```
kube-proxy
```

---

# 11. Part 1 Summary

In this chapter, you learned:

- Why Services exist
- Problems caused by changing Pod IP addresses
- What a Service is
- Service architecture
- Service discovery
- Labels and selectors
- ClusterIP
- Endpoints and EndpointSlices
- kube-proxy fundamentals

These concepts form the networking foundation of Kubernetes. In the next part, you'll learn how Services expose applications inside and outside the cluster using **NodePort**, **LoadBalancer**, **ExternalName**, and **Headless Services**.


---

# 12. Service Types

Kubernetes supports multiple Service types.

Each type is designed for a different networking requirement.

```
                Kubernetes Service

                        │

        ┌───────────────┼────────────────┐

        ▼               ▼                ▼

    ClusterIP       NodePort       LoadBalancer

                        │

                        ▼

                 ExternalName

                        │

                        ▼

                 Headless Service
```

---

# Service Type Comparison

| Service Type | Internal | External | Load Balancing | Common Use |
|--------------|----------|----------|----------------|------------|
| ClusterIP | ✅ | ❌ | ✅ | Internal Microservices |
| NodePort | ✅ | ✅ | ✅ | Testing / Labs |
| LoadBalancer | ✅ | ✅ | ✅ | Production |
| ExternalName | External DNS | External DNS | ❌ | Third-party Services |
| Headless | Internal | Optional | ❌ | Stateful Applications |

---

# 13. NodePort Service

NodePort exposes an application on a port of every Kubernetes node.

Traffic Flow

```
Internet

↓

Node IP

↓

NodePort

↓

Service

↓

Pod
```

Example

```
Node IP

192.168.1.100

↓

Port

30080

↓

Application
```

---

## Default NodePort Range

```
30000

↓

32767
```

---

## NodePort YAML

```yaml
apiVersion: v1

kind: Service

metadata:
  name: nginx-nodeport

spec:

  type: NodePort

  selector:
    app: nginx

  ports:

  - port: 80

    targetPort: 80

    nodePort: 30080
```

---

## Explanation

```
Client

↓

192.168.1.100:30080

↓

Service

↓

Container:80
```

---

## Verify

```bash
kubectl get svc
```

Example

```
NAME

nginx-nodeport

TYPE

NodePort
```

---

## Advantages

✔ Easy to configure

✔ Useful for learning

✔ No cloud provider required

---

## Limitations

❌ Fixed port range

❌ Manual port management

❌ Not ideal for Internet-facing production applications

---

# 14. LoadBalancer Service

Cloud providers can automatically provision an external load balancer.

Traffic Flow

```
Internet

↓

Cloud Load Balancer

↓

Service

↓

Pods
```

---

## Supported Platforms

- AWS
- Azure
- Google Cloud
- OCI
- DigitalOcean

---

## YAML

```yaml
apiVersion: v1

kind: Service

metadata:
  name: nginx-lb

spec:

  type: LoadBalancer

  selector:

    app: nginx

  ports:

  - port: 80

    targetPort: 80
```

---

## Verify

```bash
kubectl get svc
```

Example

```
EXTERNAL-IP

35.xx.xx.xx
```

---

## Advantages

✔ Production Ready

✔ Automatic Load Balancing

✔ Cloud Integration

---

## Limitations

❌ Usually requires a supported cloud environment

❌ May incur additional cloud costs

---

# 15. ExternalName Service

Sometimes an application inside Kubernetes needs to connect to an external service.

Instead of hardcoding the external hostname in every application,

create an ExternalName Service.

Example

```
Application

↓

ExternalName

↓

api.example.com
```

---

## YAML

```yaml
apiVersion: v1

kind: Service

metadata:

  name: external-api

spec:

  type: ExternalName

  externalName: api.example.com
```

---

## Result

Applications can simply connect to:

```
external-api
```

Kubernetes DNS resolves it to:

```
api.example.com
```

---

# Common Use Cases

- External Databases
- Third-party APIs
- SaaS Services
- Legacy Applications

---

# 16. Headless Service

Normally a Service receives a ClusterIP.

Headless Services do not.

```yaml
clusterIP: None
```

---

## Architecture

Normal Service

```
Service IP

↓

Pod
```

Headless

```
DNS

↓

Pod-1

Pod-2

Pod-3
```

Applications receive individual Pod IP addresses.

---

## Why Use Headless Services?

Useful when applications need to communicate with specific Pods.

Examples

- StatefulSets
- Databases
- Kafka
- Cassandra
- Elasticsearch

---

## YAML

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

# 17. Service Ports

One of the most confusing topics for beginners.

```
Client

↓

port

↓

Service

↓

targetPort

↓

Container
```

---

## port

The port exposed by the Service.

```yaml
port: 80
```

---

## targetPort

The container port inside the Pod.

```yaml
targetPort: 8080
```

---

## nodePort

Used only with NodePort Services.

```yaml
nodePort: 30080
```

---

## Example

```
Client

↓

30080

↓

Service Port

80

↓

Container

8080
```

This demonstrates that all three ports can be different.

---

# 18. Session Affinity

By default,

each request may go to a different Pod.

```
Request 1

↓

Pod-1

Request 2

↓

Pod-2

Request 3

↓

Pod-3
```

Sometimes an application needs a client to continue talking to the same Pod.

Enable ClientIP session affinity.

```yaml
sessionAffinity: ClientIP
```

---

## Common Use Cases

- Shopping Carts
- Legacy Applications
- Session-based Authentication

Modern applications often avoid sticky sessions by storing session data in external systems like Redis or databases.

---

# 19. External Traffic Policy

Applicable to NodePort and LoadBalancer Services.

```yaml
externalTrafficPolicy: Local
```

or

```yaml
externalTrafficPolicy: Cluster
```

---

## Cluster

Traffic may be forwarded to Pods on any node.

```
Internet

↓

Node-1

↓

Pod on Node-2
```

---

## Local

Traffic is sent only to Pods running on the node that received it.

Benefits:

- Preserves the original client source IP
- Can reduce an extra network hop

Considerations:

- If a node has no matching Pods, it cannot serve that request.

---

# 20. Common Commands

List Services

```bash
kubectl get svc
```

Describe Service

```bash
kubectl describe svc nginx
```

View Endpoints

```bash
kubectl get endpoints
```

View EndpointSlices

```bash
kubectl get endpointslices
```

Delete Service

```bash
kubectl delete svc nginx
```

---

# 21. Common Beginner Mistakes

❌ Labels do not match the Service selector.

❌ Wrong `targetPort`.

❌ Wrong `nodePort` (outside the allowed range).

❌ Expecting a LoadBalancer Service to get an external IP on a local cluster without additional software (for example, MetalLB).

❌ Forgetting that ClusterIP is only reachable from within the cluster.

---

# 22. Part 2 Summary

In this section, you learned:

- NodePort
- LoadBalancer
- ExternalName
- Headless Services
- Service ports
- Session Affinity
- External Traffic Policy
- Common networking mistakes

You now understand how Kubernetes exposes applications both internally and externally using different Service types.

---

# 23. Kubernetes DNS

Imagine you have hundreds of Pods running.

Should applications communicate using IP addresses?

```
10.244.1.15

↓

Database
```

No.

Pod IPs change frequently.

Instead, Kubernetes provides **DNS-based Service Discovery**.

Applications communicate using Service names.

Example

```
mysql

redis

backend

frontend
```

Instead of

```
10.96.15.24
```

---

# Why DNS?

Suppose a backend application needs a database.

Bad Practice

```
Database IP

↓

10.96.25.40
```

Good Practice

```
mysql.default.svc.cluster.local
```

or simply

```
mysql
```

if both applications are in the same namespace.

---

# DNS Architecture

```
Application

↓

DNS Query

↓

CoreDNS

↓

Service IP

↓

Pod
```

CoreDNS resolves the Service name into its ClusterIP.

---

# Fully Qualified Domain Name (FQDN)

Every Service has a DNS name.

Format

```
<Service Name>.<Namespace>.svc.cluster.local
```

Example

```
redis.default.svc.cluster.local
```

---

# Same Namespace

Applications can simply use

```
redis
```

---

# Different Namespace

Example

```
redis.database
```

or

```
redis.database.svc.cluster.local
```

---

# Verify DNS

Create a temporary Pod

```bash
kubectl run dns-test \
--image=busybox \
-it --rm -- sh
```

Inside the Pod

```bash
nslookup kubernetes
```

or

```bash
nslookup nginx-service
```

---

# 24. CoreDNS

CoreDNS is the default DNS server in Kubernetes.

It watches the Kubernetes API.

Whenever

- Service Created
- Service Deleted
- Namespace Created

CoreDNS automatically updates DNS records.

---

## Verify CoreDNS

```bash
kubectl get pods -n kube-system
```

Look for

```
coredns
```

---

# CoreDNS Workflow

```
Application

↓

DNS Request

↓

CoreDNS

↓

API Server

↓

Service Information

↓

Response
```

---

# 25. End-to-End Traffic Flow

Let's follow a request from a client to an application.

```
Client

↓

Service DNS

↓

CoreDNS

↓

ClusterIP

↓

kube-proxy

↓

EndpointSlice

↓

Pod

↓

Container

↓

Response
```

Each component has a specific responsibility:

- CoreDNS resolves names.
- kube-proxy implements Service networking.
- EndpointSlices identify healthy backend Pods.

---

# 26. Production Networking Best Practices

## Use Labels Consistently

Example

```yaml
labels:
  app: payment
  tier: backend
  environment: production
```

Consistent labels simplify Service selection.

---

## Use ClusterIP for Internal Communication

Examples

- Databases
- Message Brokers
- Internal APIs

Avoid exposing these directly to the Internet.

---

## Use LoadBalancer or Ingress for External Access

Instead of exposing every application with a NodePort,

use:

```
Internet

↓

Load Balancer

↓

Ingress

↓

Services

↓

Pods
```

This is the common production architecture.

---

## Avoid Hardcoding IP Addresses

Always use Service DNS names.

---

## Use Readiness Probes

Only healthy Pods should receive traffic.

Readiness probes prevent traffic from reaching Pods that are not yet ready.

---

## Monitor Service Health

Common tools

- Prometheus
- Grafana

Monitor:

- Request rate
- Error rate
- Latency

---

# 27. Service Troubleshooting

A systematic approach is essential.

---

## Step 1

Verify Service

```bash
kubectl get svc
```

---

## Step 2

Describe Service

```bash
kubectl describe svc nginx-service
```

Check:

- Type
- Selector
- Ports
- Events

---

## Step 3

Verify Matching Pods

```bash
kubectl get pods --show-labels
```

Ensure the labels match the Service selector.

---

## Step 4

Check Endpoints

```bash
kubectl get endpoints
```

or

```bash
kubectl get endpointslices
```

If no endpoints are listed,

the Service has no backend Pods.

---

## Step 5

Test DNS

```bash
nslookup nginx-service
```

---

## Step 6

Test Connectivity

```bash
kubectl exec -it POD_NAME -- curl http://nginx-service
```

---

# Common Problems

## No Endpoints

Cause

```
Labels

≠

Selectors
```

Solution

Correct the labels or selector.

---

## Connection Refused

Possible causes

- Incorrect targetPort
- Application not listening
- Container crashed

---

## DNS Failure

Check

```bash
kubectl get pods -n kube-system
```

Verify that CoreDNS is running.

---

## LoadBalancer Pending

Possible causes

- Cloud provider integration unavailable
- Local cluster without MetalLB

---

## NodePort Not Reachable

Check

- Firewall
- Security Groups
- Correct node IP
- Correct nodePort

---

# 28. Hands-on Labs

## Lab 1

Create a Deployment

```bash
kubectl create deployment nginx --image=nginx
```

---

## Lab 2

Expose it internally

```bash
kubectl expose deployment nginx \
--port=80
```

Verify

```bash
kubectl get svc
```

---

## Lab 3

Convert to NodePort

Edit the Service

```yaml
type: NodePort
```

Verify

```bash
kubectl get svc
```

---

## Lab 4

Check DNS

```bash
kubectl run dns-test \
--image=busybox \
-it --rm -- sh
```

Run

```bash
nslookup nginx
```

---

## Lab 5

Delete one Pod

```bash
kubectl delete pod POD_NAME
```

Observe:

- Service remains available
- New Pod joins automatically
- EndpointSlice updates

---

# 29. CKA Exam Tips

✔ Understand every Service type.

✔ Know the difference between:

- port
- targetPort
- nodePort

✔ Troubleshoot using:

```bash
kubectl describe svc
```

✔ Always verify Endpoints or EndpointSlices.

✔ Practice DNS testing using BusyBox.

✔ Remember that Services route traffic only to Pods matching their selectors.

---

# 30. Interview Questions

## Basic

1. What is a Kubernetes Service?
2. Why do Pods need Services?
3. What is ClusterIP?
4. What is NodePort?
5. What is LoadBalancer?

---

## Intermediate

1. Explain Service discovery.
2. Difference between ClusterIP and Headless Service.
3. Difference between Endpoints and EndpointSlices.
4. Explain kube-proxy.
5. Explain ExternalName.

---

## Advanced

1. Explain the complete Service networking flow.
2. How does CoreDNS work?
3. How does kube-proxy route traffic?
4. Explain sessionAffinity.
5. Explain externalTrafficPolicy.

---

## Scenario-Based

### Scenario 1

Your Service exists, but no traffic reaches the application.

What checks would you perform?

---

### Scenario 2

A Service has no endpoints.

What is the most likely cause?

---

### Scenario 3

A LoadBalancer Service remains in the `Pending` state.

How would you investigate?

---

### Scenario 4

Pods are healthy, but DNS resolution fails.

Which Kubernetes component should you examine first?

---

### Scenario 5

A NodePort Service is inaccessible from outside the cluster.

List the networking checks you would perform.

---

# 31. Service Cheat Sheet

Create Service

```bash
kubectl expose deployment nginx --port=80
```

List Services

```bash
kubectl get svc
```

Describe Service

```bash
kubectl describe svc nginx
```

List Endpoints

```bash
kubectl get endpoints
```

List EndpointSlices

```bash
kubectl get endpointslices
```

List Pods

```bash
kubectl get pods --show-labels
```

Test DNS

```bash
nslookup nginx
```

Delete Service

```bash
kubectl delete svc nginx
```

---

# 32. Key Takeaways

- Services provide stable networking for ephemeral Pods.
- Clients communicate with Services instead of Pod IPs.
- Labels and selectors connect Services to backend Pods.
- ClusterIP is the default internal Service type.
- NodePort and LoadBalancer expose applications externally.
- ExternalName maps Kubernetes Services to external DNS names.
- Headless Services expose individual Pod identities.
- CoreDNS provides Service discovery.
- EndpointSlices improve scalability for Service backends.
- kube-proxy implements Service networking on cluster nodes.

---

# 33. Chapter Summary

Congratulations! 🎉

You have completed one of the most important networking topics in Kubernetes.

In this chapter, you learned:

- Why Services exist
- Service architecture
- ClusterIP
- NodePort
- LoadBalancer
- ExternalName
- Headless Services
- Labels and selectors
- Service ports
- Session affinity
- External traffic policy
- DNS and CoreDNS
- EndpointSlices
- kube-proxy
- Service troubleshooting
- Production best practices
- Hands-on labs
- CKA preparation
- Interview questions

You now have a solid understanding of how Kubernetes provides reliable communication between applications, even as Pods are created, updated, and deleted.

This knowledge prepares you for the next major networking topic: **Ingress**, where you'll learn how to expose multiple applications through a single entry point using host-based and path-based routing.