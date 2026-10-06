# K8S-18 - Environment Variables, Downward API & Service Discovery (Part 1)

> **Class Date:** 10-Jul-2026
>
> **Module:** Application Configuration


---

# 📚 Table of Contents

1. Introduction
2. Why Dynamic Configuration?
3. Environment Variables
4. env vs envFrom
5. ConfigMaps with Environment Variables
6. Secrets with Environment Variables
7. First Deployment Example
8. Part 1 Summary

---

# 1. Introduction

Modern applications should be portable.

Avoid hardcoding:

- Database IP addresses
- API endpoints
- Credentials
- Namespace names
- Environment names

Instead, applications should receive configuration from Kubernetes.

---

# Traditional Approach

```
Application

↓

Database IP Hardcoded

↓

10.0.0.15

↓

Database Changed

↓

Application Fails
```

---

# Kubernetes Approach

```
Application

↓

Environment Variables

↓

Database Service

↓

Works Across Environments
```

---

# 2. Why Dynamic Configuration?

Imagine three environments:

```
Development

↓

db-dev

----------------

Staging

↓

db-stage

----------------

Production

↓

db-prod
```

The application code should remain unchanged.

Only the configuration changes.

---

# Benefits

- Portability
- Reusability
- Easier deployments
- GitOps friendly
- Cloud native
- Better security

---

# 3. Environment Variables

The simplest way to pass configuration into a container.

Example:

```yaml
env:

- name: APP_ENV

  value: production
```

Inside the container:

```
APP_ENV=production
```

---

## Multiple Variables

```yaml
env:

- name: APP_NAME

  value: payment

- name: LOG_LEVEL

  value: info

- name: REGION

  value: ap-south-1
```

---

## Verify

```bash
kubectl exec -it pod-name -- env
```

---

# Architecture

```
Deployment

↓

Environment Variables

↓

Container

↓

Application Reads Variables
```

---

# 4. env vs envFrom

Kubernetes provides two ways to inject configuration.

---

## env

Import selected values.

Example:

```yaml
env:

- name: LOG_LEVEL

  value: info
```

or

```yaml
env:

- name: DB_HOST

  valueFrom:

    configMapKeyRef:

      name: app-config

      key: database
```

---

## envFrom

Import all keys from a ConfigMap or Secret.

Example:

```yaml
envFrom:

- configMapRef:

    name: app-config
```

---

# Comparison

| Feature | env | envFrom |
|----------|-----|----------|
| Import one variable | ✅ | ❌ |
| Import all keys | ❌ | ✅ |
| Rename variable | ✅ | ❌ |
| Fine-grained control | ✅ | Limited |

---

# 5. ConfigMaps with Environment Variables

ConfigMap:

```yaml
apiVersion: v1

kind: ConfigMap

metadata:

  name: app-config

data:

  APP_ENV: production

  LOG_LEVEL: info

  DATABASE_HOST: payment-db
```

---

Deployment:

```yaml
envFrom:

- configMapRef:

    name: app-config
```

---

Result:

```
APP_ENV

LOG_LEVEL

DATABASE_HOST
```

are automatically available inside the container.

---

# 6. Secrets with Environment Variables

Sensitive values should come from Secrets.

Secret:

```yaml
apiVersion: v1

kind: Secret

metadata:

  name: db-secret

type: Opaque

stringData:

  DB_USER: admin

  DB_PASSWORD: password123
```

---

Deployment:

```yaml
envFrom:

- secretRef:

    name: db-secret
```

---

Inside the container:

```
DB_USER

DB_PASSWORD
```

---

## Best Practice

- Use ConfigMaps for **non-sensitive** configuration.
- Use Secrets for **sensitive** information.
- Consider external secret managers (Vault, AWS Secrets Manager, Azure Key Vault, Google Secret Manager) for production environments.

---

# 7. First Deployment Example

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: payment

spec:

  replicas: 2

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

        env:

        - name: APP_ENV

          value: production

        - name: REGION

          value: ap-south-1

        envFrom:

        - configMapRef:

            name: app-config

        - secretRef:

            name: db-secret
```

---

# YAML Explained

## env

Defines individual variables.

---

## envFrom

Imports all keys from referenced ConfigMaps or Secrets.

---

## Result

Application receives:

```
APP_ENV

REGION

APP_CONFIG values

SECRET values
```

without changing application code.

---

# Verify

```bash
kubectl exec -it <pod-name> -- printenv
```

or

```bash
kubectl exec -it <pod-name> -- env
```

---

# 8. Part 1 Summary

You learned:

- Why dynamic configuration matters
- Environment variables
- `env`
- `envFrom`
- ConfigMaps with environment variables
- Secrets with environment variables
- Production Deployment example

You now understand how Kubernetes injects configuration into applications without hardcoding values.

# K8S-18 - Environment Variables, Downward API & Service Discovery (Part 2)

> **Class Date:** 10-Jul-2026

---

# 📚 Table of Contents

9. Downward API
10. fieldRef
11. resourceFieldRef
12. Downward API Volumes
13. Service Discovery
14. Kubernetes DNS
15. DNS Search Paths
16. Production Configuration Patterns
17. Part 2 Summary

---

# 9. Downward API

The **Downward API** allows a Pod to access information about itself without calling the Kubernetes API.

Examples:

- Pod name
- Namespace
- Pod IP
- Node name
- Labels
- Annotations
- Resource requests and limits

---

## Why?

Instead of hardcoding:

```
Application

↓

Knows Nothing About Itself
```

Use the Downward API:

```
Pod Metadata

↓

Environment Variable

↓

Application
```

---

# 10. fieldRef

`fieldRef` exposes Pod fields as environment variables.

Example:

```yaml
env:
- name: POD_NAME
  valueFrom:
    fieldRef:
      fieldPath: metadata.name

- name: POD_NAMESPACE
  valueFrom:
    fieldRef:
      fieldPath: metadata.namespace

- name: NODE_NAME
  valueFrom:
    fieldRef:
      fieldPath: spec.nodeName

- name: POD_IP
  valueFrom:
    fieldRef:
      fieldPath: status.podIP
```

---

## Result

Inside the container:

```
POD_NAME=payment-7c9d5
POD_NAMESPACE=production
NODE_NAME=worker-2
POD_IP=10.244.1.15
```

---

## Common fieldPath Values

| fieldPath | Description |
|-----------|-------------|
| `metadata.name` | Pod name |
| `metadata.namespace` | Namespace |
| `metadata.uid` | Unique Pod ID |
| `spec.nodeName` | Node hosting the Pod |
| `status.podIP` | Pod IP address |
| `status.hostIP` | Node IP address |

---

# 11. resourceFieldRef

`resourceFieldRef` exposes container resource values.

Example:

```yaml
env:
- name: CPU_REQUEST
  valueFrom:
    resourceFieldRef:
      resource: requests.cpu

- name: MEMORY_LIMIT
  valueFrom:
    resourceFieldRef:
      resource: limits.memory
```

---

## Result

```
CPU_REQUEST=500m
MEMORY_LIMIT=1Gi
```

Applications can log or adjust behavior based on their allocated resources.

---

# 12. Downward API Volumes

Metadata can also be exposed as files.

Example:

```yaml
volumes:
- name: pod-info
  downwardAPI:
    items:
    - path: "labels"
      fieldRef:
        fieldPath: metadata.labels

    - path: "annotations"
      fieldRef:
        fieldPath: metadata.annotations
```

Mount the volume:

```yaml
volumeMounts:
- name: pod-info
  mountPath: /etc/podinfo
  readOnly: true
```

---

## Result

```
/etc/podinfo/

├── labels
└── annotations
```

Applications can read these files directly.

---

# 13. Service Discovery

Pods should communicate using **Service names**, not Pod IPs.

Incorrect:

```
10.244.1.15
```

Correct:

```
payment-db
```

---

## Architecture

```
Frontend Pod

↓

payment-db

↓

Kubernetes DNS

↓

ClusterIP

↓

Database Pods
```

If database Pods restart, the Service name remains the same.

---

# 14. Kubernetes DNS

Every Service receives a DNS name.

Example Service:

```yaml
metadata:
  name: payment-db
```

Within the same namespace:

```
payment-db
```

Across namespaces:

```
payment-db.production
```

Fully qualified domain name (FQDN):

```
payment-db.production.svc.cluster.local
```

---

## DNS Resolution Flow

```
Application

↓

payment-db

↓

CoreDNS

↓

ClusterIP

↓

Backend Pods
```

---

# 15. DNS Search Paths

Pods automatically receive DNS search domains.

Example `/etc/resolv.conf`:

```text
search production.svc.cluster.local svc.cluster.local cluster.local
```

This allows:

```
payment-db
```

to resolve automatically when the Service is in the same namespace.

---

# 16. Production Configuration Patterns

## Pattern 1

Non-sensitive configuration:

```
ConfigMap

↓

envFrom
```

---

## Pattern 2

Sensitive configuration:

```
Secret

↓

envFrom
```

---

## Pattern 3

Pod metadata:

```
Downward API

↓

fieldRef
```

---

## Pattern 4

Service communication:

```
Application

↓

Service Name

↓

DNS

↓

Backend
```

---

## Complete Example

```yaml
env:
- name: POD_NAME
  valueFrom:
    fieldRef:
      fieldPath: metadata.name

- name: POD_NAMESPACE
  valueFrom:
    fieldRef:
      fieldPath: metadata.namespace

envFrom:
- configMapRef:
    name: app-config

- secretRef:
    name: db-secret
```

---

# YAML Explained

### fieldRef

Injects Pod metadata.

---

### envFrom

Imports ConfigMap and Secret values.

---

### Service Discovery

Use Service DNS names instead of Pod IP addresses.

---

# 17. Part 2 Summary

You learned:

- Downward API
- `fieldRef`
- `resourceFieldRef`
- Downward API volumes
- Service discovery
- Kubernetes DNS
- DNS search paths
- Production configuration patterns

You now understand how Kubernetes enables applications to discover their own metadata and communicate with other services dynamically.

# K8S-18 - Environment Variables, Downward API & Service Discovery (Part 3)

> **Class Date:** 10-Jul-2026

---

# 📚 Table of Contents

18. Common Configuration Mistakes
19. Troubleshooting
20. Production Best Practices
21. Hands-on Labs
22. CKA Exam Tips
23. Interview Questions
24. Cheat Sheet
25. Chapter Summary

---

# 18. Common Configuration Mistakes

---

## Mistake 1: Hardcoding Pod IPs

❌ Bad

```
Database

↓

10.244.1.35
```

Problem:

- Pod IP changes after restart.
- Application configuration becomes invalid.

✅ Good

```
payment-db
```

Always communicate through a Kubernetes Service.

---

## Mistake 2: Storing Secrets in ConfigMaps

❌

```yaml
data:
  DB_PASSWORD: mypassword
```

Problem:

ConfigMaps are intended for **non-sensitive** configuration.

✅

Use:

```yaml
kind: Secret
```

---

## Mistake 3: Hardcoding Environment Names

Avoid:

```yaml
APP_ENV=production
```

inside the application source code.

Instead:

```
ConfigMap

↓

Environment Variable

↓

Application
```

This allows the same image to run in development, staging, and production.

---

## Mistake 4: Depending on Pod Names

Avoid writing application logic that assumes a specific Pod name.

Pod names can change when Deployments create replacement Pods.

If stable identities are required, consider a StatefulSet.

---

## Mistake 5: Calling the Kubernetes API Unnecessarily

Applications often only need:

- Pod name
- Namespace
- Node name

Use the Downward API instead of querying the Kubernetes API for these values.

---

# 19. Troubleshooting

---

## Show Environment Variables

```bash
kubectl exec -it <pod-name> -- env
```

or

```bash
kubectl exec -it <pod-name> -- printenv
```

---

## Inspect ConfigMap

```bash
kubectl describe configmap app-config
```

---

## Inspect Secret

```bash
kubectl describe secret db-secret
```

To view Secret values (base64 encoded):

```bash
kubectl get secret db-secret -o yaml
```

---

## Verify Downward API

```bash
kubectl exec -it <pod-name> -- env
```

Look for:

```
POD_NAME
POD_NAMESPACE
NODE_NAME
POD_IP
```

---

## Verify Downward API Volume

```bash
kubectl exec -it <pod-name> -- ls /etc/podinfo
```

Read file contents:

```bash
kubectl exec -it <pod-name> -- cat /etc/podinfo/labels
```

---

## Test DNS Resolution

Start a temporary debugging Pod:

```bash
kubectl run dns-test \
  --image=busybox:1.36 \
  --restart=Never \
  -it --rm -- sh
```

Inside the Pod:

```bash
nslookup payment-db
```

or, if available:

```bash
wget -qO- http://payment-api:8080/health
```

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

> **Note:** In modern Kubernetes, `EndpointSlice` is the primary mechanism used internally for endpoint management, but `kubectl get endpoints` remains useful for quick troubleshooting.

---

## Inspect EndpointSlices

```bash
kubectl get endpointslices
```

---

# Common Problems

| Problem | Possible Cause | Solution |
|----------|----------------|----------|
| Variable missing | Wrong ConfigMap/Secret key | Verify key names |
| Pod starts with old values | ConfigMap updated after startup | Restart Pods if values are consumed only as environment variables |
| DNS lookup fails | Service missing or wrong namespace | Verify Service name and namespace |
| Connection refused | Service exists but no healthy backend Pods | Check selectors and Pod readiness |
| Downward API value empty | Incorrect `fieldPath` | Verify supported field paths |

---

# 20. Production Best Practices

---

## Separate Configuration

Keep:

- Application code
- Configuration
- Secrets

independent.

---

## Use Service Names

Always connect to:

```
payment-db
```

instead of:

```
10.244.x.x
```

---

## Prefer ConfigMaps for Non-Sensitive Data

Examples:

- Log level
- Feature flags
- API URLs
- Region

---

## Prefer Secrets for Sensitive Data

Examples:

- Passwords
- API tokens
- Certificates
- Database credentials

For enterprise environments, integrate with external secret managers where appropriate.

---

## Use Downward API

Inject runtime metadata such as:

- Pod name
- Namespace
- Node name

without requiring Kubernetes API access.

---

## Keep Images Environment-Agnostic

Build one container image.

Deploy it to:

- Development
- Testing
- Staging
- Production

using different configuration resources.

---

# 21. Hands-on Labs

---

## Lab 1

Create a ConfigMap.

Inject values using:

```yaml
envFrom
```

Verify:

```bash
env
```

inside the container.

---

## Lab 2

Create a Secret.

Inject credentials.

Verify environment variables.

---

## Lab 3

Inject:

```yaml
metadata.name
metadata.namespace
status.podIP
```

using `fieldRef`.

Display the values from within the container.

---

## Lab 4

Expose labels through a Downward API volume.

Verify:

```bash
cat /etc/podinfo/labels
```

---

## Lab 5

Create:

- Frontend Deployment
- Backend Deployment
- Backend Service

Access the backend using the Service DNS name instead of a Pod IP.

---

# 22. CKA Exam Tips

✔ Know the difference:

- `env`
- `envFrom`
- ConfigMap
- Secret
- Downward API

✔ Memorize:

- `fieldRef`
- `resourceFieldRef`

✔ Practice:

```bash
kubectl exec
kubectl describe configmap
kubectl describe secret
kubectl get svc
kubectl get endpoints
kubectl get endpointslices
```

✔ Remember:

- Services provide stable network identities.
- Pod IPs are ephemeral.

---

# 23. Interview Questions

## Beginner

1. What is a ConfigMap?
2. What is a Secret?
3. What is `envFrom`?
4. What is the Downward API?
5. Why should applications use Service names?

---

## Intermediate

1. Compare `env` and `envFrom`.
2. Explain `fieldRef`.
3. Explain `resourceFieldRef`.
4. Why are Pod IPs not suitable for application configuration?
5. What is the Kubernetes Service FQDN?

---

## Advanced

1. Design a configuration strategy for a multi-environment application.
2. Explain how Downward API improves security and portability.
3. Compare ConfigMaps, Secrets, and external secret managers.
4. How would you troubleshoot DNS failures in a Kubernetes cluster?
5. How would you design configuration for a large microservices platform?

---

## Scenario-Based

### Scenario 1

An application cannot connect to `payment-db`.

Which Kubernetes resources would you inspect first?

---

### Scenario 2

A developer updates a ConfigMap, but the application still uses the old value.

Why might this happen?

---

### Scenario 3

A Pod must know its own namespace and name.

Which Kubernetes feature would you use?

---

### Scenario 4

A workload must connect to another service running in a different namespace.

What DNS name should it use?

---

### Scenario 5

An application contains hardcoded production URLs.

How would you redesign the configuration?

---

# 24. Cheat Sheet

Show environment variables:

```bash
kubectl exec -it <pod-name> -- env
```

Inspect ConfigMap:

```bash
kubectl describe configmap <name>
```

Inspect Secret:

```bash
kubectl describe secret <name>
```

List Services:

```bash
kubectl get svc
```

List Endpoints:

```bash
kubectl get endpoints
```

List EndpointSlices:

```bash
kubectl get endpointslices
```

Test DNS:

```bash
nslookup <service-name>
```

Show Pod metadata:

```bash
kubectl get pod <pod-name> -o yaml
```

---

# 25. Chapter Summary

Congratulations! 🎉

You have completed **K8S-18 – Environment Variables, Downward API & Service Discovery**.

In this chapter, you learned:

- Environment variables
- `env` vs `envFrom`
- ConfigMaps
- Secrets
- Downward API
- `fieldRef`
- `resourceFieldRef`
- Downward API volumes
- Service discovery
- Kubernetes DNS
- DNS search paths
- Troubleshooting
- Hands-on labs
- Production best practices
- CKA preparation
- Interview questions

You now understand how Kubernetes applications receive configuration, discover services, and access runtime metadata without relying on hardcoded values.