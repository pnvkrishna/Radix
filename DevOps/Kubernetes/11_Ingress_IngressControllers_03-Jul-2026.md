# K8S-11 - Ingress & Ingress Controllers (Part 1)

> **Class Date:** 03-Jul-2026
>
> **Module:** Ingress & Ingress Controllers
>
> **File Name:** `K8S-11_Ingress_IngressControllers_03-Jul-2026.md`

---

# 📚 Table of Contents

1. Introduction
2. Why Ingress?
3. Problems with NodePort & LoadBalancer
4. What is Ingress?
5. What is an Ingress Controller?
6. Ingress Architecture
7. Request Flow
8. NGINX Ingress Controller
9. Installing an Ingress Controller
10. Summary

---

# 1. Introduction

Applications deployed inside Kubernetes are usually exposed using Services.

Example:

```
Frontend

↓

Service

↓

ClusterIP
```

But ClusterIP is only accessible **inside** the cluster.

To make applications available to users on the Internet,

Kubernetes needs another layer.

That layer is **Ingress**.

---

# 2. Why Ingress?

Imagine an e-commerce application.

```
Frontend

Inventory

Payments

Orders

Authentication

Reports
```

If every application uses its own LoadBalancer,

```
Frontend

↓

LoadBalancer

-----------------

Inventory

↓

LoadBalancer

-----------------

Payments

↓

LoadBalancer
```

Problems:

- High cloud cost
- Many public IPs
- Difficult management
- Duplicate TLS configuration

---

# Better Architecture

```
Internet

↓

Load Balancer

↓

Ingress Controller

↓

Services

↓

Pods
```

One entry point can route traffic to many applications.

---

# 3. Problems with NodePort & LoadBalancer

## NodePort

```
NodeIP:30080

NodeIP:30081

NodeIP:30082
```

Problems:

- High port numbers
- Difficult to remember
- No hostname routing
- No TLS termination
- Poor user experience

---

## LoadBalancer

```
Frontend

↓

Public IP

-------------------

Inventory

↓

Public IP

-------------------

Payments

↓

Public IP
```

Problems:

- Expensive
- Many public IP addresses
- Hard to manage certificates
- Difficult to centralize routing rules

---

# 4. What is Ingress?

Ingress is a Kubernetes API resource that defines **HTTP and HTTPS routing rules**.

An Ingress resource itself does **not** handle traffic.

It describes:

- Which hostnames should be served
- Which URL paths should be matched
- Which backend Services should receive requests

An **Ingress Controller** reads these rules and implements them.

---

# Architecture

```
Internet

↓

Ingress

↓

Service

↓

Pod
```

More accurately:

```
Ingress Resource

↓

Ingress Controller

↓

Service

↓

Pod
```

---

# Ingress Responsibilities

✔ Host-based routing

✔ Path-based routing

✔ TLS termination

✔ URL rewrites (controller-dependent)

✔ Centralized traffic management

---

# 5. What is an Ingress Controller?

An Ingress Controller is the software that watches Ingress resources and configures a reverse proxy or load balancer accordingly.

Without an Ingress Controller:

```
Ingress YAML

↓

Nothing Happens
```

Because Ingress resources are only configuration.

---

# Workflow

```
Ingress YAML

↓

API Server

↓

Ingress Controller

↓

Reverse Proxy Configuration

↓

Traffic Routing
```

---

# Popular Ingress Controllers

- NGINX Ingress Controller
- HAProxy Ingress
- Traefik
- Kong
- F5 NGINX Ingress Controller
- AWS Load Balancer Controller (integrates with AWS load balancers)
- Azure Application Gateway Ingress Controller (AGIC)

Each has different features, but they all implement the idea of routing traffic based on Ingress resources.

---

# 6. Ingress Architecture

```
                 Internet

                      │

                      ▼

              Cloud Load Balancer

                      │

                      ▼

            Ingress Controller

                      │

      ┌───────────────┼───────────────┐

      ▼               ▼               ▼

   frontend-svc   payment-svc   inventory-svc

      │               │               │

      ▼               ▼               ▼

     Pods            Pods            Pods
```

---

# Benefits

✔ Single public entry point

✔ Centralized routing

✔ Lower cloud cost

✔ Easier certificate management

✔ Scalable architecture

---

# 7. Request Flow

Example request:

```
https://shop.example.com/products
```

Flow:

```
Browser

↓

DNS

↓

Load Balancer

↓

Ingress Controller

↓

Service

↓

Pod

↓

Response
```

Each component has a specific role:

- DNS resolves the hostname.
- Load Balancer forwards traffic to the cluster.
- Ingress Controller applies routing rules.
- Service load balances across Pods.
- Pod processes the request.

---

# 8. NGINX Ingress Controller

The **NGINX Ingress Controller** is one of the most widely used Ingress Controllers.

It:

- Watches Ingress resources.
- Generates NGINX configuration.
- Reloads NGINX when routing rules change.

Architecture:

```
Ingress Resource

↓

NGINX Ingress Controller

↓

NGINX Configuration

↓

Traffic Routing
```

---

# Features

✔ Host-based routing

✔ Path-based routing

✔ TLS termination

✔ Rate limiting (via controller features)

✔ URL rewrites

✔ Load balancing

✔ Health checking (controller-dependent)

---

# 9. Installing an Ingress Controller

An Ingress Controller is **not automatically installed** in every Kubernetes cluster.

Install it according to your platform.

Examples:

- Managed Kubernetes add-ons
- Helm charts
- Official deployment manifests

After installation:

```bash
kubectl get pods -n ingress-nginx
```

Example output:

```
NAME

ingress-nginx-controller
```

Verify the Service:

```bash
kubectl get svc -n ingress-nginx
```

Depending on the platform, the controller Service may be:

- LoadBalancer
- NodePort
- ClusterIP (behind another load balancer)

---

# Verify IngressClass

```bash
kubectl get ingressclass
```

Example:

```
NAME

nginx
```

The `IngressClass` identifies which controller should process an Ingress resource.

---

# 10. Part 1 Summary

In this section, you learned:

- Why Ingress exists
- Problems with NodePort
- Problems with LoadBalancer
- What an Ingress resource is
- What an Ingress Controller does
- Ingress architecture
- Request flow
- NGINX Ingress Controller
- Basic installation concepts
- IngressClass

You now understand the relationship between Ingress resources, Ingress Controllers, Services, and Pods.

# K8S-11 - Ingress & Ingress Controllers (Part 2)

---

# 📚 Table of Contents

11. First Ingress Resource
12. Anatomy of an Ingress YAML
13. Host-Based Routing
14. Path-Based Routing
15. pathType Explained
16. Default Backend
17. Multiple Services Behind One Ingress
18. Production Examples
19. Common Mistakes
20. Part 2 Summary

---

# 11. First Ingress Resource

Let's expose a web application.

Architecture:

```
Internet

↓

Ingress Controller

↓

web-service

↓

web-pod
```

---

## Example Ingress

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:
  name: web-ingress

spec:

  ingressClassName: nginx

  rules:

  - host: app.example.com

    http:

      paths:

      - path: /

        pathType: Prefix

        backend:

          service:

            name: web-service

            port:

              number: 80
```

---

# Apply

```bash
kubectl apply -f ingress.yaml
```

Verify

```bash
kubectl get ingress
```

---

# 12. Anatomy of an Ingress YAML

## apiVersion

```yaml
apiVersion: networking.k8s.io/v1
```

Current stable API.

---

## kind

```yaml
kind: Ingress
```

Creates an Ingress resource.

---

## ingressClassName

```yaml
ingressClassName: nginx
```

Specifies which Ingress Controller should process this resource.

---

## host

```yaml
host: app.example.com
```

Matches requests for:

```
https://app.example.com
```

---

## path

```yaml
path: /
```

Matches the URL path.

---

## backend

```yaml
backend:

  service:

    name: web-service

    port:

      number: 80
```

Requests matching the rule are forwarded to:

```
web-service:80
```

---

# 13. Host-Based Routing

One Ingress can serve multiple hostnames.

```
shop.example.com

↓

shop-service

-------------------

api.example.com

↓

api-service

-------------------

admin.example.com

↓

admin-service
```

---

## Example

```yaml
rules:

- host: shop.example.com

  http:

    paths:

    - path: /

      pathType: Prefix

      backend:

        service:

          name: shop-service

          port:

            number: 80

- host: api.example.com

  http:

    paths:

    - path: /

      pathType: Prefix

      backend:

        service:

          name: api-service

          port:

            number: 80
```

---

# Request Flow

```
shop.example.com

↓

Ingress Controller

↓

shop-service

↓

Pods
```

---

# 14. Path-Based Routing

Different URL paths can route to different Services.

Example:

```
example.com/

↓

frontend

-------------------

example.com/api

↓

api-service

-------------------

example.com/admin

↓

admin-service
```

---

## Example YAML

```yaml
rules:

- host: example.com

  http:

    paths:

    - path: /

      pathType: Prefix

      backend:

        service:

          name: frontend

          port:

            number: 80

    - path: /api

      pathType: Prefix

      backend:

        service:

          name: api-service

          port:

            number: 8080

    - path: /admin

      pathType: Prefix

      backend:

        service:

          name: admin-service

          port:

            number: 80
```

---

# Architecture

```
example.com

           │

───────────┼────────────

           │

     /          /api        /admin

     │            │             │

frontend     api-service   admin-service
```

---

# 15. pathType Explained

Every path rule requires a `pathType`.

---

## Prefix

```yaml
pathType: Prefix
```

Matches:

```
/

/products

/products/123

/products/cart
```

Most commonly used.

---

## Exact

```yaml
pathType: Exact
```

Matches only:

```
/login
```

Does **not** match:

```
/login/

/login/admin
```

---

## ImplementationSpecific

Behavior depends on the Ingress Controller.

Avoid using it unless you understand your controller's implementation.

---

# Comparison

| pathType | Matches |
|-----------|----------|
| Prefix | Path and subpaths |
| Exact | Exact path only |
| ImplementationSpecific | Controller-defined |

---

# 16. Default Backend

What if no rule matches?

```
example.com/random

↓

No Matching Rule

↓

Default Backend
```

The response is commonly:

```
404 Not Found
```

Some controllers allow configuring a custom default backend to return branded error pages or maintenance messages.

---

# 17. Multiple Services Behind One Ingress

Real production example:

```
Internet

↓

Ingress Controller

↓

────────────────────────────

│

├── frontend

├── users

├── orders

├── payments

├── inventory

├── reports

└── notifications
```

One Ingress Controller can route requests to many Services.

---

# Enterprise Example

```
company.com

↓

/

↓

Frontend

-----------------------

company.com/api

↓

Backend API

-----------------------

company.com/auth

↓

Authentication

-----------------------

company.com/admin

↓

Admin Portal
```

---

# 18. Production Examples

## Microservices

```
shop.example.com

↓

Frontend

------------

shop.example.com/api

↓

Backend

------------

shop.example.com/payment

↓

Payment Service
```

---

## Multi-Team Platform

```
hr.company.com

↓

HR

-------------------

finance.company.com

↓

Finance

-------------------

support.company.com

↓

Support
```

Each hostname routes independently while sharing the same Ingress Controller.

---

# 19. Common Mistakes

## Wrong Service Name

```
Backend Service

↓

Not Found
```

Always verify:

```bash
kubectl get svc
```

---

## Wrong Port

Example:

```
Ingress → 8080

Service → 80
```

Traffic fails because the backend port is incorrect.

---

## Missing IngressClass

If multiple controllers exist,

the Ingress may not be processed.

Always verify:

```bash
kubectl get ingressclass
```

---

## Missing DNS Record

```
app.example.com

↓

DNS Missing

↓

Request Fails
```

Ingress does not create public DNS records.

DNS must point to the LoadBalancer or gateway address used by the Ingress Controller.

---

## Controller Not Installed

```
Ingress YAML

↓

No Controller

↓

No Traffic
```

Always confirm the controller Pods are healthy.

---

# Useful Commands

List Ingresses:

```bash
kubectl get ingress
```

Describe an Ingress:

```bash
kubectl describe ingress web-ingress
```

Check Services:

```bash
kubectl get svc
```

Check Controller Pods:

```bash
kubectl get pods -n ingress-nginx
```

---

# 20. Part 2 Summary

In this section, you learned:

- How to create an Ingress
- Ingress YAML structure
- Host-based routing
- Path-based routing
- `pathType`
- Default backend
- Multiple Services behind one Ingress
- Real production routing examples
- Common configuration mistakes
- Useful troubleshooting commands

You can now route multiple applications through a single Ingress Controller using hostnames and URL paths.

# K8S-11 - Ingress & Ingress Controllers (Part 3)

> **Class Date:** 03-Jul-2026

---

# 📚 Table of Contents

21. TLS & HTTPS
22. TLS Secrets
23. cert-manager & Let's Encrypt
24. Common Ingress Annotations
25. Authentication & Rate Limiting
26. Gateway API Introduction
27. Ingress Troubleshooting
28. Hands-on Labs
29. CKA Exam Tips
30. Interview Questions
31. Ingress Cheat Sheet
32. Chapter Summary

---

# 21. TLS & HTTPS

Without TLS:

```
Browser

↓

HTTP

↓

Plain Text

↓

Internet
```

Problems:

- Passwords can be intercepted
- Session cookies exposed
- Data is unencrypted

---

With TLS:

```
Browser

↓

HTTPS

↓

Encrypted Traffic

↓

Ingress Controller

↓

Service

↓

Pods
```

TLS protects data while it travels over the network.

---

## TLS Termination

Most production clusters terminate TLS at the Ingress Controller.

```
Internet

↓

HTTPS

↓

Ingress Controller

↓

HTTP

↓

Service

↓

Pod
```

Benefits:

✔ Centralized certificate management

✔ Reduced application complexity

✔ Easier certificate renewal

---

# 22. TLS Secrets

Certificates are stored as Kubernetes Secrets.

Example:

```yaml
tls:

- hosts:

  - app.example.com

  secretName: app-tls
```

---

## Create TLS Secret

```bash
kubectl create secret tls app-tls \
  --cert=tls.crt \
  --key=tls.key
```

Verify:

```bash
kubectl get secret app-tls
```

Type:

```
kubernetes.io/tls
```

---

## Complete TLS Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress

metadata:
  name: web-ingress

spec:

  ingressClassName: nginx

  tls:

  - hosts:

    - app.example.com

    secretName: app-tls

  rules:

  - host: app.example.com

    http:

      paths:

      - path: /

        pathType: Prefix

        backend:

          service:

            name: web-service

            port:

              number: 80
```

---

# 23. cert-manager & Let's Encrypt

Managing certificates manually does not scale.

Modern Kubernetes commonly uses:

```
Let's Encrypt

↓

cert-manager

↓

TLS Secret

↓

Ingress
```

---

## How It Works

```
Ingress

↓

cert-manager

↓

Certificate Request

↓

Let's Encrypt

↓

Certificate Issued

↓

TLS Secret Created

↓

Automatic Renewal
```

---

## Benefits

✔ Automatic renewal

✔ Free certificates

✔ Less operational effort

✔ Production-ready

---

# 24. Common Ingress Annotations

Annotations provide controller-specific features.

Example:

```yaml
metadata:

  annotations:

    nginx.ingress.kubernetes.io/rewrite-target: /
```

---

## Common NGINX Annotations

### URL Rewrite

```yaml
nginx.ingress.kubernetes.io/rewrite-target
```

---

### Force HTTPS

```yaml
nginx.ingress.kubernetes.io/ssl-redirect: "true"
```

---

### Request Size Limit

```yaml
nginx.ingress.kubernetes.io/proxy-body-size: 50m
```

Useful for file uploads.

---

### Connection Timeouts

```yaml
nginx.ingress.kubernetes.io/proxy-read-timeout
```

Useful for long-running API requests.

---

> **Note:** Annotation names and supported features vary by Ingress Controller. Always consult your controller's documentation.

---

# 25. Authentication & Rate Limiting

Ingress Controllers often support security features.

---

## Basic Authentication

```
Browser

↓

Username

↓

Password

↓

Ingress

↓

Application
```

---

## OAuth / OIDC

Common enterprise integrations:

- Microsoft Entra ID (Azure AD)
- Google Identity
- Okta
- Keycloak

Authentication is handled before requests reach the application.

---

## Rate Limiting

Protects applications from excessive traffic.

```
Client

↓

100 Requests/Second

↓

Ingress

↓

Limit Applied

↓

Application Protected
```

Common use cases:

- Prevent abuse
- Reduce brute-force attacks
- Protect backend resources

---

# 26. Gateway API Introduction

Ingress works well,

but it has limitations.

The Kubernetes community introduced the **Gateway API** to provide more flexibility and richer traffic management.

---

## Traditional Ingress

```
Internet

↓

Ingress

↓

Service
```

---

## Gateway API

```
Gateway

↓

HTTPRoute

↓

Service
```

Gateway API separates responsibilities:

- Infrastructure teams manage Gateways.
- Application teams manage Routes.

---

## Gateway API Resources

| Resource | Purpose |
|-----------|----------|
| GatewayClass | Defines the Gateway implementation |
| Gateway | Entry point into the cluster |
| HTTPRoute | HTTP routing rules |
| TCPRoute | TCP routing |
| TLSRoute | TLS traffic |
| GRPCRoute | gRPC routing |

---

## Why Gateway API?

✔ More expressive routing

✔ Better multi-team separation

✔ Standardized policy attachment

✔ Designed for future networking needs

> **Current Reality:** Ingress remains widely deployed today, while Gateway API adoption continues to grow. Knowing both is valuable.

---

# 27. Ingress Troubleshooting

## Step 1

Check Ingress.

```bash
kubectl get ingress
```

---

## Step 2

Describe Ingress.

```bash
kubectl describe ingress web-ingress
```

Look for:

- Events
- Rules
- Backend Services
- TLS configuration

---

## Step 3

Check Services.

```bash
kubectl get svc
```

Ensure the Service exists and exposes the expected port.

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

No endpoints usually means the Service selector is not matching any Pods.

---

## Step 5

Check Controller Logs

```bash
kubectl logs deployment/ingress-nginx-controller \
-n ingress-nginx
```

---

## Step 6

Verify DNS

```bash
nslookup app.example.com
```

or

```bash
dig app.example.com
```

The hostname should resolve to the LoadBalancer IP or hostname.

---

# Common Problems

| Problem | Possible Cause | Solution |
|----------|----------------|----------|
| 404 Not Found | No matching rule | Check host/path |
| 503 Service Unavailable | No backend endpoints | Verify Pods and Service selectors |
| TLS Error | Missing or invalid certificate | Check TLS Secret |
| DNS Failure | Incorrect DNS record | Update DNS |
| Ingress Ignored | Wrong IngressClass | Verify `ingressClassName` |

---

# 28. Hands-on Labs

## Lab 1

Deploy:

- Frontend
- API
- Admin

Create one Ingress routing all three.

---

## Lab 2

Configure host-based routing.

```
shop.example.com

api.example.com

admin.example.com
```

---

## Lab 3

Configure path-based routing.

```
/

/api

/admin
```

---

## Lab 4

Enable HTTPS using a TLS Secret.

Verify:

```bash
curl -vk https://app.example.com
```

---

## Lab 5

Install cert-manager.

Create a ClusterIssuer.

Request a certificate.

Verify automatic Secret creation and renewal.

---

# 29. CKA Exam Tips

✔ Know:

- Ingress
- Ingress Controller
- IngressClass

✔ Practice:

```bash
kubectl get ingress
kubectl describe ingress
```

✔ Understand:

- Host-based routing
- Path-based routing
- TLS configuration

✔ Remember:

Ingress resources do **not** process traffic by themselves.

---

# 30. Interview Questions

## Beginner

1. What is Ingress?
2. Why use Ingress instead of NodePort?
3. What is an Ingress Controller?
4. What is IngressClass?
5. Explain path-based routing.

---

## Intermediate

1. Explain host-based routing.
2. How is TLS configured?
3. What is cert-manager?
4. How do annotations work?
5. Explain default backends.

---

## Advanced

1. Compare Ingress and Gateway API.
2. Explain TLS termination.
3. How would you secure a production Ingress?
4. How would you troubleshoot HTTP 503 errors?
5. How would you design ingress for hundreds of microservices?

---

## Scenario-Based

### Scenario 1

Users receive HTTP 404.

How would you investigate?

---

### Scenario 2

TLS works for one hostname but not another.

What would you verify?

---

### Scenario 3

A Service is healthy,

but Ingress returns HTTP 503.

Where would you look?

---

### Scenario 4

Your company wants automatic certificate renewal.

Which Kubernetes components would you recommend?

---

### Scenario 5

Multiple teams need independent routing rules without managing shared infrastructure.

Would Gateway API help? Why?

---

# 31. Ingress Cheat Sheet

List Ingresses:

```bash
kubectl get ingress
```

Describe:

```bash
kubectl describe ingress
```

List IngressClasses:

```bash
kubectl get ingressclass
```

Check TLS Secrets:

```bash
kubectl get secret
```

Check Services:

```bash
kubectl get svc
```

Check Endpoints:

```bash
kubectl get endpointslices
```

View Controller Pods:

```bash
kubectl get pods -n ingress-nginx
```

View Controller Logs:

```bash
kubectl logs deployment/ingress-nginx-controller \
-n ingress-nginx
```

---

# 32. Key Takeaways

- Ingress provides HTTP/HTTPS routing.
- An Ingress Controller implements Ingress resources.
- One Ingress Controller can serve many applications.
- Host-based and path-based routing reduce infrastructure cost.
- TLS is typically terminated at the Ingress Controller.
- cert-manager automates certificate issuance and renewal.
- Annotations enable controller-specific features.
- Gateway API is the next-generation Kubernetes networking API.
- DNS, Services, Endpoints, and IngressClass are common troubleshooting areas.

---

# 33. Chapter Summary

Congratulations! 🎉

You have completed **K8S-11 – Ingress & Ingress Controllers**.

In this chapter, you learned:

- Why Ingress exists
- Ingress architecture
- Ingress Controllers
- NGINX Ingress Controller
- IngressClass
- Host-based routing
- Path-based routing
- `pathType`
- Default backends
- TLS & HTTPS
- TLS Secrets
- cert-manager
- Common annotations
- Authentication & rate limiting
- Gateway API fundamentals
- Troubleshooting
- Hands-on labs
- CKA preparation
- Interview questions

You now have the knowledge to expose Kubernetes applications securely and efficiently in production environments.