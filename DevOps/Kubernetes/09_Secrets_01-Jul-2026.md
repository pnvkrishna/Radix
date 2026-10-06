# K8S-09 - Kubernetes Secrets (Part 1)

> **Class Date:** 01-Jul-2026
>
> **Module:** Kubernetes Secrets


---

# 📚 Table of Contents

1. Introduction
2. What are Secrets?
3. Why Secrets?
4. ConfigMaps vs Secrets
5. Secret Architecture
6. Secret Types
7. Creating Secrets
8. Secret YAML
9. Base64 Encoding Explained
10. Security Considerations
11. Summary

---

# 1. Introduction

Almost every application needs sensitive information to function.

Examples include:

- Database passwords
- API Keys
- OAuth Tokens
- SSH Keys
- TLS Certificates
- Docker Registry Credentials
- Cloud Access Keys

Hardcoding these values into source code or container images is a major security risk.

Kubernetes provides **Secrets** to store and distribute sensitive information more securely.

---

# 2. What is a Secret?

A Secret is a Kubernetes API object used to store **sensitive information**.

Examples:

```
DATABASE_PASSWORD

API_KEY

JWT_SECRET

AWS_ACCESS_KEY

TLS_CERTIFICATE

SSH_PRIVATE_KEY
```

Applications retrieve these values at runtime instead of embedding them into code.

---

# Real World Example

Think of a company office.

Everyone can read the notice board.

↓

Equivalent to

```
ConfigMap
```

Only authorized employees can open the company safe.

↓

Equivalent to

```
Secret
```

---

# 3. Why Secrets?

Suppose a developer writes:

```python
PASSWORD = "MyPassword123"
```

Problems:

- Visible in Git
- Visible in Docker images
- Visible during code review
- Difficult to rotate
- Security compliance violations

Instead

```
Application

↓

Reads Secret

↓

Authenticates

↓

Runs
```

The application never needs the password inside its source code.

---

# 4. ConfigMap vs Secret

| Feature | ConfigMap | Secret |
|----------|-----------|---------|
| Purpose | Configuration | Sensitive Data |
| Passwords | ❌ | ✅ |
| API Keys | ❌ | ✅ |
| TLS Certificates | ❌ | ✅ |
| Environment Variables | ✅ | ✅ |
| Volume Mount | ✅ | ✅ |
| Kubernetes Resource | ✅ | ✅ |

---

## Important Clarification

A Secret is **not automatically encrypted**.

By default:

- Kubernetes stores Secret data as **Base64-encoded** values.
- Base64 is **encoding**, not encryption.
- Encryption at rest must be explicitly enabled on the cluster to protect stored Secret data in the backing datastore.

We'll discuss encryption at rest later in this chapter and in advanced security topics.

---

# 5. Secret Architecture

```
                 Secret

        +----------------------+
        | PASSWORD=********    |
        | API_KEY=********     |
        | TOKEN=********       |
        +----------------------+

                  │

                  ▼

            Kubernetes API

                  │

                  ▼

                 Pod

                  │

                  ▼

            Application
```

Applications consume Secrets without hardcoding sensitive values.

---

# 6. Secret Types

Kubernetes provides several Secret types.

---

## 1. Opaque (Default)

Most commonly used.

Stores arbitrary key-value pairs.

Example

```
USERNAME

PASSWORD

TOKEN
```

---

## 2. kubernetes.io/dockerconfigjson

Stores container registry credentials.

Commonly used for:

- Docker Hub
- GitHub Container Registry (GHCR)
- Amazon ECR
- Azure Container Registry (ACR)
- Google Artifact Registry (GAR)

---

## 3. kubernetes.io/tls

Stores:

- TLS Certificate
- Private Key

Used by:

- Ingress Controllers
- HTTPS Applications

---

## 4. kubernetes.io/ssh-auth

Stores:

```
SSH Private Key
```

Useful for Git access and SSH authentication.

---

## 5. kubernetes.io/basic-auth

Stores:

- Username
- Password

---

## 6. kubernetes.io/service-account-token

Historically used for ServiceAccount tokens.

Modern Kubernetes versions typically use **short-lived tokens** obtained through the **TokenRequest API** rather than relying on long-lived Secret-based tokens.

---

# 7. Creating Secrets

## Method 1 - Imperative Command

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=MyPassword123
```

---

## Verify

```bash
kubectl get secrets
```

Example

```
NAME

db-secret
```

---

## Describe

```bash
kubectl describe secret db-secret
```

---

## YAML

```bash
kubectl get secret db-secret -o yaml
```

---

# 8. Secret YAML

Example

```yaml
apiVersion: v1

kind: Secret

metadata:
  name: db-secret

type: Opaque

stringData:
  username: admin
  password: MyPassword123
```

---

## Why stringData?

`stringData` allows you to write plaintext values.

When the Secret is created:

```
stringData

↓

API Server

↓

Base64 Encoding

↓

Stored as data
```

This makes manifests easier to read and maintain.

---

## Stored Form

When retrieved:

```yaml
data:
  username: YWRtaW4=
  password: TXlQYXNzd29yZDEyMw==
```

---

# data vs stringData

| Feature | data | stringData |
|----------|------|------------|
| Base64 Required | ✅ | ❌ |
| Easier to Read | ❌ | ✅ |
| Sent to API | ✅ | Converted Automatically |

For authoring YAML, `stringData` is usually more convenient.

---

# 9. Base64 Encoding Explained

Example

```
admin
```

becomes

```
YWRtaW4=
```

Example

```
password123
```

becomes

```
cGFzc3dvcmQxMjM=
```

---

## Encode

Linux

```bash
echo -n "admin" | base64
```

PowerShell

```powershell
[Convert]::ToBase64String(
    [System.Text.Encoding]::UTF8.GetBytes("admin")
)
```

---

## Decode

Linux

```bash
echo "YWRtaW4=" | base64 --decode
```

PowerShell

```powershell
[System.Text.Encoding]::UTF8.GetString(
    [Convert]::FromBase64String("YWRtaW4=")
)
```

---

# Important Warning

Base64 is **not encryption**.

Anyone who has access to the encoded value can decode it easily.

Example:

```
YWRtaW4=

↓

Decode

↓

admin
```

Base64 is used because Kubernetes stores Secret data as bytes in the API object format—not because it provides confidentiality.

---

# 10. Security Considerations

✔ Never commit plaintext Secrets to Git repositories.

✔ Use RBAC to restrict who can read Secret objects.

✔ Enable encryption at rest for Secrets in the cluster.

✔ Rotate credentials regularly.

✔ Prefer external secret management systems for production environments.

✔ Audit access to Secrets.

---

# 11. Part 1 Summary

In this chapter, you learned:

- What Secrets are
- Why Secrets exist
- ConfigMaps vs Secrets
- Secret architecture
- Secret types
- Creating Secrets
- Secret YAML
- `data` vs `stringData`
- Base64 encoding
- Fundamental security considerations

You now understand how Kubernetes stores sensitive information and why Secrets should be used instead of ConfigMaps for confidential data.

---

# 12. Using Secrets in Pods

Creating a Secret is only the first step.

Applications must consume Secrets securely.

Kubernetes supports multiple methods.

```
                    Secret

                       │

       ┌───────────────┼────────────────┐

       ▼               ▼                ▼

Environment        Environment        Volume
 Variable           Variables         Mount
(secretKeyRef)      (envFrom)

```

Each method has different advantages.

---

# 13. Method 1 – Environment Variables

A Secret value can be exposed as an environment variable.

## Example Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  username: admin
  password: MyPassword123
```

---

## Pod Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-client
spec:
  containers:
  - name: app
    image: nginx

    env:
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username

    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

---

## Workflow

```
Secret

↓

secretKeyRef

↓

Environment Variable

↓

Application
```

---

## Verify

```bash
kubectl exec -it mysql-client -- env
```

Example

```
DB_USERNAME=admin
DB_PASSWORD=********
```

> **Note:** The application can read the actual values. Masking is shown here only for illustration.

---

# Advantages

✔ Easy

✔ Explicit

✔ Good for small numbers of secrets

---

# Limitations

❌ Secrets become part of the process environment inside the container.

❌ Applications usually need a restart to pick up updated environment variables.

---

# 14. Method 2 – envFrom

Instead of importing one key at a time,

import every key.

```yaml
envFrom:

- secretRef:

    name: db-secret
```

---

## Workflow

```
Secret

↓

All Keys

↓

Environment Variables

↓

Application
```

---

## Example

Secret

```
username=admin

password=MyPassword123
```

Application automatically receives

```
username

password
```

---

# Advantages

✔ Less YAML

✔ Easy to maintain

---

# Limitation

Imports every key.

Not ideal if only one value is needed.

---

# env vs envFrom

| Feature | env | envFrom |
|----------|-----|----------|
| Individual Keys | ✅ | ❌ |
| Entire Secret | ❌ | ✅ |
| Explicit | ✅ | ❌ |
| Smaller YAML | ❌ | ✅ |

---

# 15. Method 3 – Mount Secret as Volume

Many applications expect configuration files rather than environment variables.

Kubernetes can mount a Secret as files.

---

## Architecture

```
Secret

↓

Volume

↓

Files

↓

Application
```

---

## Pod YAML

```yaml
volumes:

- name: secret-volume

  secret:

    secretName: db-secret
```

Container

```yaml
volumeMounts:

- name: secret-volume

  mountPath: /etc/secrets

  readOnly: true
```

---

## Result

```
/etc/secrets

↓

username

password
```

Each key becomes a separate file.

---

## Verify

```bash
kubectl exec -it mysql-client -- ls /etc/secrets
```

Read

```bash
cat /etc/secrets/username
```

Output

```
admin
```

---

# Why Use Volume Mounts?

Many applications read secrets from files.

Examples

- NGINX TLS certificates
- PostgreSQL SSL keys
- Java keystores
- SSH private keys

Volume mounts fit naturally with these applications.

---

# 16. Secret Update Behavior

This is a common interview topic.

---

## Environment Variables

```
Secret

↓

Pod Starts

↓

Environment Variables Loaded

↓

Secret Updated

↓

Running Process Still Uses Old Values
```

A Pod restart is typically required.

---

## Mounted Secret Volume

```
Secret

↓

Mounted Files

↓

Secret Updated

↓

Mounted Files Updated
```

Kubernetes refreshes the mounted files after a short synchronization interval.

Whether the application uses the new values depends on whether it reloads the files.

---

# Production Recommendation

If the application does **not** support hot reload:

```
Secret Updated

↓

Rolling Restart

↓

New Pods

↓

New Secret Loaded
```

---

Restart Deployment

```bash
kubectl rollout restart deployment payment
```

---

# 17. Image Pull Secrets

Private container registries require authentication.

Examples:

- Docker Hub (private repositories)
- GitHub Container Registry
- Amazon ECR
- Azure Container Registry
- Google Artifact Registry

---

## Create Image Pull Secret

```bash
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=myuser \
  --docker-password=mypassword
```

---

## Use in Pod

```yaml
spec:
  imagePullSecrets:
    - name: regcred
```

---

## Workflow

```
Pod

↓

ImagePullSecret

↓

Registry Authentication

↓

Container Image Downloaded
```

---

# 18. ServiceAccounts and ImagePullSecrets

Instead of defining `imagePullSecrets` in every Pod,

attach it to a ServiceAccount.

```
ServiceAccount

↓

imagePullSecrets

↓

All Pods Using That ServiceAccount
```

Benefits

✔ Less duplication

✔ Easier maintenance

✔ Consistent authentication

---

# 19. Common Mistakes

❌ Using ConfigMaps for passwords.

❌ Forgetting `readOnly: true` for Secret volumes.

❌ Committing Secret manifests with plaintext values to Git.

❌ Expecting environment variables to refresh automatically.

❌ Wrong Secret name.

❌ Wrong key.

❌ Missing namespace.

---

# 20. Production Best Practices

### Principle of Least Privilege

Only Pods that need a Secret should have access to it.

---

### Keep Secrets Small

Separate Secrets by application.

Example

```
payment-secret

inventory-secret

auth-secret
```

---

### Prefer Volume Mounts

For certificates and key files,

volume mounts are usually cleaner than environment variables.

---

### Rotate Credentials

Regularly rotate:

- Database passwords
- API Keys
- Tokens
- Certificates

---

### Use External Secret Managers

In production,

many organizations store secrets outside Kubernetes.

Examples include:

- HashiCorp Vault
- AWS Secrets Manager
- Azure Key Vault
- Google Secret Manager

Kubernetes retrieves them through dedicated integrations.

---

# 21. Real Production Example

E-commerce Application

```
Frontend

↓

Inventory

↓

Payment

↓

Database
```

Each service has its own Secret.

```
inventory-secret

payment-secret

database-secret
```

Advantages

✔ Better isolation

✔ Easier rotation

✔ Improved access control

---

# 22. Useful Commands

List Secrets

```bash
kubectl get secrets
```

Describe

```bash
kubectl describe secret db-secret
```

View YAML

```bash
kubectl get secret db-secret -o yaml
```

Decode a value (Linux)

```bash
kubectl get secret db-secret \
-o jsonpath='{.data.username}' \
| base64 --decode
```

Edit

```bash
kubectl edit secret db-secret
```

Delete

```bash
kubectl delete secret db-secret
```

---

# 23. Part 2 Summary

In this section, you learned:

- Using Secrets with `secretKeyRef`
- Using `envFrom`
- Mounting Secrets as volumes
- Secret update behavior
- Image Pull Secrets
- ServiceAccounts with imagePullSecrets
- Production best practices
- Common mistakes

You now understand how applications securely consume Secrets in Kubernetes and how to choose the right approach for different workloads.


# K8S-09 - Kubernetes Secrets (Part 3)

---

# 📚 Table of Contents

24. Encryption at Rest
25. RBAC and Secret Access
26. External Secret Management
27. Secret Rotation
28. GitOps and Sealed Secrets
29. Secret Troubleshooting
30. Hands-on Labs
31. CKA Exam Tips
32. Interview Questions
33. Secret Cheat Sheet
34. Chapter Summary

---

# 24. Encryption at Rest

One of the biggest misconceptions is:

> "Kubernetes Secrets are encrypted by default."

That is **not universally true**.

### By Default

```
Application

↓

Secret

↓

API Server

↓

etcd
```

Secrets are stored in `etcd`.

If **encryption at rest is not configured**, Secret values are not protected by storage encryption provided by Kubernetes. Base64 encoding alone does **not** provide confidentiality.

---

## Encryption at Rest

When enabled:

```
Secret

↓

API Server

↓

Encryption Provider

↓

Encrypted Data

↓

etcd
```

Benefits:

- Protects stored Secret data.
- Helps meet security and compliance requirements.
- Reduces risk if the datastore is compromised.

---

## Best Practice

✔ Enable encryption at rest for production clusters.

✔ Restrict direct access to `etcd`.

---

# 25. RBAC and Secret Access

Secrets should never be readable by everyone.

Use **Role-Based Access Control (RBAC)**.

Example:

```
Developer

↓

Role

↓

Read ConfigMaps

❌ No Secret Access
```

Operations team:

```
Operations

↓

Role

↓

Read Secrets

✔ Allowed
```

---

## Principle of Least Privilege

Grant only the permissions required.

Avoid broad permissions such as:

```yaml
resources:
- secrets
verbs:
- "*"
```

Instead, grant only the specific verbs and resources that are necessary.

---

# 26. External Secret Management

Many organizations avoid storing long-lived secrets directly in Kubernetes.

Instead:

```
Application

↓

Kubernetes

↓

External Secret Operator

↓

Secret Manager

↓

Cloud Provider
```

This allows credentials to remain in a centralized secret management system.

---

## Popular Secret Managers

### HashiCorp Vault

Features:

- Dynamic credentials
- Secret rotation
- Audit logging
- Fine-grained access control

---

### AWS Secrets Manager

Common for Amazon EKS.

Stores:

- Database credentials
- API keys
- Certificates

Supports automatic rotation for many integrations.

---

### Azure Key Vault

Common for Azure Kubernetes Service (AKS).

Stores:

- Keys
- Secrets
- Certificates

Integrates with Azure identity services.

---

### Google Secret Manager

Common for Google Kubernetes Engine (GKE).

Provides centralized secret storage with IAM-based access control.

---

## External Secrets Operator

A popular pattern is:

```
AWS Secrets Manager

↓

External Secrets Operator

↓

Kubernetes Secret

↓

Pod
```

The operator synchronizes external secrets into Kubernetes.

---

# 27. Secret Rotation

Passwords should not remain unchanged forever.

Rotation process:

```
Old Secret

↓

Create New Credential

↓

Update Secret

↓

Rollout

↓

Application Uses New Credential

↓

Remove Old Credential
```

---

## What Should Be Rotated?

- Database passwords
- API keys
- TLS certificates
- OAuth tokens
- SSH keys

---

## Production Recommendation

Automate secret rotation wherever supported.

Manual rotation is error-prone.

---

# 28. GitOps and Sealed Secrets

A common question:

> "Can I store Kubernetes Secrets in Git?"

Not in plaintext.

---

## Problem

```
Git Repository

↓

Secret.yaml

↓

Password Visible
```

This is a security risk.

---

## Sealed Secrets

Tools such as **Bitnami Sealed Secrets** allow encrypted Secret manifests to be stored in Git.

Workflow:

```
Developer

↓

Seal Secret

↓

Encrypted Manifest

↓

Git

↓

Cluster

↓

Controller

↓

Kubernetes Secret
```

Benefits:

- Git-friendly
- Encrypted for storage in the repository
- Fits GitOps workflows

---

## Other GitOps Approaches

Some teams use:

- External Secrets Operator
- Vault integrations
- Cloud-native secret managers

The best choice depends on organizational requirements and infrastructure.

---

# 29. Secret Troubleshooting

## Step 1

Verify the Secret exists.

```bash
kubectl get secrets
```

---

## Step 2

Describe it.

```bash
kubectl describe secret db-secret
```

---

## Step 3

Verify namespace.

```bash
kubectl get secrets -A
```

---

## Step 4

Verify Pod events.

```bash
kubectl describe pod payment
```

Look for:

```
Secret not found
```

or

```
Couldn't find key
```

---

## Step 5

Verify mounted files.

```bash
kubectl exec -it payment -- ls /etc/secrets
```

---

## Step 6

Verify environment variables.

```bash
kubectl exec -it payment -- env
```

Remember:

Sensitive values may be visible inside the container environment. Handle this output carefully.

---

# Common Problems

| Problem | Cause | Solution |
|---------|-------|----------|
| Secret not found | Wrong namespace | Verify namespace |
| Wrong key | Typo | Check Secret keys |
| Application still using old value | Environment variables don't auto-refresh | Restart workload |
| Image pull failed | Missing imagePullSecret | Verify registry credentials |
| Permission denied | RBAC | Review Roles and RoleBindings |

---

# 30. Hands-on Labs

## Lab 1

Create a Secret.

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=MyPassword123
```

---

## Lab 2

Expose the Secret as environment variables.

Verify:

```bash
kubectl exec -it POD_NAME -- env
```

---

## Lab 3

Mount the Secret as a volume.

Read:

```bash
cat /etc/secrets/password
```

---

## Lab 4

Create an imagePullSecret.

Attach it to a Pod or ServiceAccount.

Verify that the image pulls successfully from a private registry.

---

## Lab 5

Update the Secret.

Observe:

- Mounted Secret files update after Kubernetes refreshes them.
- Environment variables require Pod recreation to reflect changes.

---

# 31. CKA Exam Tips

✔ Know the difference between ConfigMaps and Secrets.

✔ Understand:

- `secretKeyRef`
- `envFrom`
- Secret volumes

✔ Practice:

```bash
kubectl create secret generic
```

✔ Know how to inspect:

```bash
kubectl describe secret
```

✔ Understand imagePullSecrets.

✔ Remember that Base64 encoding is **not encryption**.

---

# 32. Interview Questions

## Beginner

1. What is a Kubernetes Secret?
2. Why use Secrets instead of ConfigMaps?
3. What is Base64 encoding?
4. Is Base64 encryption?
5. What are common Secret types?

---

## Intermediate

1. Explain `secretKeyRef`.
2. Explain Secret volume mounts.
3. What is an imagePullSecret?
4. How do Secret updates behave?
5. Explain `stringData` vs `data`.

---

## Advanced

1. Explain encryption at rest.
2. How would you secure Secret access using RBAC?
3. Compare Vault and cloud-native secret managers.
4. Explain External Secrets Operator.
5. How would you rotate secrets without downtime?

---

## Scenario-Based

### Scenario 1

A Pod reports:

```
Secret not found
```

How would you troubleshoot it?

---

### Scenario 2

A private container image fails to pull.

What would you check?

---

### Scenario 3

A Secret changed, but the application still uses the old value.

Why?

---

### Scenario 4

Your company uses GitOps.

How would you safely manage Secrets in Git?

---

### Scenario 5

An auditor asks how Secrets are protected in your Kubernetes cluster.

What controls would you describe?

---

# 33. Secret Cheat Sheet

Create:

```bash
kubectl create secret generic db-secret \
  --from-literal=username=admin \
  --from-literal=password=MyPassword123
```

List:

```bash
kubectl get secrets
```

Describe:

```bash
kubectl describe secret db-secret
```

View YAML:

```bash
kubectl get secret db-secret -o yaml
```

Decode a value (Linux):

```bash
kubectl get secret db-secret \
-o jsonpath='{.data.password}' \
| base64 --decode
```

Delete:

```bash
kubectl delete secret db-secret
```

Restart Deployment:

```bash
kubectl rollout restart deployment payment
```

---

# 34. Key Takeaways

- Secrets store sensitive information.
- Base64 encoding is **not** encryption.
- Enable encryption at rest for production clusters.
- Use RBAC to limit access.
- Prefer external secret managers in enterprise environments.
- Rotate credentials regularly.
- Store encrypted or externally managed secrets in GitOps workflows.
- Use `stringData` when authoring Secret manifests.
- Separate Secret ownership by application and environment.

---

# 35. Chapter Summary

Congratulations! 🎉

You have completed **K8S-09 – Kubernetes Secrets**.

In this chapter, you learned:

- What Secrets are
- ConfigMaps vs Secrets
- Secret types
- Creating Secrets
- `data` vs `stringData`
- Base64 encoding
- Environment variables
- Secret volume mounts
- Image Pull Secrets
- Secret update behavior
- Encryption at rest
- RBAC
- External secret management
- Secret rotation
- GitOps and Sealed Secrets
- Troubleshooting
- Hands-on labs
- CKA preparation
- Interview questions

You now have a strong foundation in managing sensitive information securely in Kubernetes. This knowledge prepares you for the next topic: **Volumes & Persistent Storage**, where you'll learn how Kubernetes handles persistent data for stateful applications.