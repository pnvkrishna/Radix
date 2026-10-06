# Kubernetes Class 08 - ConfigMaps (Part 1)

> **Class Date:** 30-Jun-2026
>
> **Module:** ConfigMaps


---

# 📚 Table of Contents

1. Introduction
2. What is Configuration?
3. Why ConfigMaps?
4. Problems with Hardcoded Configuration
5. What is a ConfigMap?
6. ConfigMap Architecture
7. Creating ConfigMaps
8. Creating ConfigMaps from Files
9. ConfigMap YAML Explained
10. Summary

---

# 1. Introduction

Modern applications require configuration to run correctly.

Examples include:

- Database host
- Database port
- Application name
- Environment (Dev, QA, Production)
- Feature flags
- API endpoints
- Log levels
- Cache configuration

A common mistake is storing these values directly inside the application code.

Kubernetes solves this problem using **ConfigMaps**.

ConfigMaps allow configuration to be stored separately from application code, making applications easier to maintain and deploy across different environments.

---

# 2. What is Configuration?

Configuration is any value that controls how an application behaves without changing its source code.

Examples:

```
Application

↓

Database Host

↓

mysql.default.svc.cluster.local

----------------------

Environment

↓

Production

----------------------

Log Level

↓

INFO

----------------------

Port

↓

8080
```

Changing configuration should not require rebuilding the application image.

---

# Real World Example

Imagine a mobile phone.

The phone hardware remains the same.

Only the settings change.

Examples:

- Language
- Brightness
- Wi-Fi
- Wallpaper

Configuration changes behavior without changing the device itself.

Applications work in a similar way.

---

# 3. Why ConfigMaps?

Suppose a developer writes:

```python
DATABASE_HOST = "10.10.10.25"
```

This creates several problems.

If the database IP changes:

- Source code must be modified.
- The application must be rebuilt.
- A new container image must be created.
- A new deployment must be released.

This is inefficient and error-prone.

---

# Better Approach

Application

↓

Reads Configuration

↓

ConfigMap

↓

Runs Normally

Now configuration changes do not require changing application code.

---

# 4. Problems with Hardcoded Configuration

Consider three environments.

```
Development

↓

Database

↓

dev-db

------------------

Testing

↓

Database

↓

test-db

------------------

Production

↓

Database

↓

prod-db
```

Without ConfigMaps, you might create separate images for each environment.

Problems:

- Multiple container images
- Duplicate deployments
- Difficult maintenance
- Higher risk of configuration errors

---

# Twelve-Factor App Principle

One of the core ideas of cloud-native application design is:

> **Store configuration outside the application code.**

ConfigMaps help Kubernetes applications follow this principle.

---

# 5. What is a ConfigMap?

A ConfigMap is a Kubernetes resource used to store **non-sensitive configuration data** as key-value pairs.

Examples:

```text
APP_NAME=Inventory

APP_PORT=8080

LOG_LEVEL=INFO

ENVIRONMENT=production
```

Applications can read these values at runtime.

---

## Important Rule

Use ConfigMaps only for **non-sensitive** information.

Examples:

✔ Application name

✔ Port numbers

✔ URLs

✔ Feature flags

Do **not** store:

- Passwords
- API Keys
- Tokens
- Certificates

Those belong in **Secrets**, covered in the next chapter.

---

# 6. ConfigMap Architecture

```
              ConfigMap

        +------------------+
        | APP_NAME=Store   |
        | PORT=8080        |
        | LOG_LEVEL=INFO   |
        +------------------+

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

The application retrieves configuration from the ConfigMap rather than embedding it in code.

---

# 7. Creating a ConfigMap

## Imperative Method

```bash
kubectl create configmap app-config \
  --from-literal=APP_NAME=Inventory \
  --from-literal=APP_PORT=8080 \
  --from-literal=LOG_LEVEL=INFO
```

---

## Verify

```bash
kubectl get configmaps
```

Example output:

```
NAME

app-config
```

---

## View Details

```bash
kubectl describe configmap app-config
```

---

## YAML Output

```bash
kubectl get configmap app-config -o yaml
```

---

# 8. Creating ConfigMaps from Files

Suppose you have a configuration file:

```properties
APP_NAME=Inventory

APP_PORT=8080

LOG_LEVEL=INFO
```

Save it as:

```
app.properties
```

Create the ConfigMap:

```bash
kubectl create configmap app-config \
  --from-file=app.properties
```

Kubernetes stores the file contents in the ConfigMap.

This approach is useful for larger configuration files.

---

# 9. ConfigMap YAML

Example:

```yaml
apiVersion: v1

kind: ConfigMap

metadata:
  name: app-config

data:
  APP_NAME: Inventory
  APP_PORT: "8080"
  LOG_LEVEL: INFO
```

---

## Line-by-Line Explanation

### apiVersion

```yaml
v1
```

Defines the API version.

---

### kind

```yaml
ConfigMap
```

Specifies the Kubernetes resource type.

---

### metadata

Stores metadata such as:

- Name
- Labels
- Annotations

---

### data

Contains the configuration values.

Example:

```yaml
data:
  APP_NAME: Inventory
  APP_PORT: "8080"
  LOG_LEVEL: INFO
```

Values in `data` are stored as strings.

---

# Apply YAML

```bash
kubectl apply -f configmap.yaml
```

---

# Verify

```bash
kubectl get configmaps
```

---

# Edit ConfigMap

```bash
kubectl edit configmap app-config
```

---

# Delete ConfigMap

```bash
kubectl delete configmap app-config
```

---

# 10. Part 1 Summary

In this chapter, you learned:

- What configuration is
- Why external configuration is important
- Problems with hardcoded values
- The purpose of ConfigMaps
- ConfigMap architecture
- Creating ConfigMaps using commands
- Creating ConfigMaps from files
- ConfigMap YAML explained line by line

You now understand how Kubernetes separates application configuration from application code, making deployments more portable, maintainable, and cloud-native.

---

# 11. Using ConfigMaps in Pods

Creating a ConfigMap is only the first step.

The real power of ConfigMaps comes from using them inside Pods.

Kubernetes supports multiple ways to consume ConfigMaps.

```
                 ConfigMap

                      │

      ┌───────────────┼────────────────┐

      ▼               ▼                ▼

 Environment      Environment       Volume

 Variable          Variables        Mount
   (env)           (envFrom)

```

Each approach has different use cases.

---

# 12. Method 1 - Environment Variables (env)

This is the simplest approach.

Individual ConfigMap values are mapped to environment variables.

Example ConfigMap

```yaml
data:
  APP_NAME: Inventory
  APP_PORT: "8080"
```

---

## Pod YAML

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: inventory

spec:

  containers:

  - name: app

    image: nginx

    env:

    - name: APP_NAME

      valueFrom:

        configMapKeyRef:

          name: app-config

          key: APP_NAME
```

---

## Workflow

```
ConfigMap

↓

APP_NAME

↓

Environment Variable

↓

Application
```

---

## Verify

```bash
kubectl exec -it inventory -- env
```

Output

```
APP_NAME=Inventory
```

---

# Advantages

✔ Easy

✔ Explicit

✔ Only required values are imported

---

# Limitations

❌ Every variable must be defined separately.

Large ConfigMaps become repetitive.

---

# 13. Method 2 - envFrom

Instead of mapping every key,

import the entire ConfigMap.

---

## Example

```yaml
envFrom:

- configMapRef:

    name: app-config
```

---

## Workflow

```
ConfigMap

↓

All Keys

↓

Environment Variables

↓

Application
```

---

## Example

ConfigMap

```text
APP_NAME=Inventory

PORT=8080

LOG_LEVEL=INFO
```

Application receives

```
APP_NAME

PORT

LOG_LEVEL
```

Automatically.

---

# Advantages

✔ Less YAML

✔ Easy to maintain

✔ Good for medium-sized applications

---

# Limitations

❌ Imports every key.

Not suitable when only a few values are required.

---

# env vs envFrom

| Feature | env | envFrom |
|---------|-----|---------|
| Single Key | ✅ | ❌ |
| Entire ConfigMap | ❌ | ✅ |
| Explicit | ✅ | ❌ |
| Less YAML | ❌ | ✅ |

---

# 14. Method 3 - ConfigMap as Volume

This is one of the most powerful methods.

Instead of environment variables,

Kubernetes creates files.

---

## Architecture

```
ConfigMap

↓

Volume

↓

Files

↓

Application
```

---

## Example YAML

```yaml
volumes:

- name: config-volume

  configMap:

    name: app-config
```

---

Container

```yaml
volumeMounts:

- name: config-volume

  mountPath: /etc/config
```

---

## Result

```
/etc/config

↓

APP_NAME

APP_PORT

LOG_LEVEL
```

Each key becomes a file.

---

## Verify

```bash
kubectl exec -it inventory -- ls /etc/config
```

Example

```
APP_NAME

APP_PORT

LOG_LEVEL
```

---

Read

```bash
cat /etc/config/APP_NAME
```

Output

```
Inventory
```

---

# Why Mount as Files?

Many applications already expect configuration files.

Examples

- NGINX
- Apache
- Prometheus
- Grafana
- MySQL

Instead of changing the application,

Kubernetes provides configuration as files.

---

# 15. ConfigMap Update Behavior

This is an important interview topic.

---

## Environment Variables

Application starts.

↓

Reads ConfigMap.

↓

Stores values.

If ConfigMap changes later

↓

Application **does not automatically receive updated environment variables**.

The Pod typically needs to be restarted to pick up the new values.

---

## Volume Mount

Application reads configuration file.

↓

ConfigMap updated.

↓

Mounted files are updated automatically by Kubernetes (typically within a short synchronization interval).

Whether the application uses the new values immediately depends on whether it reloads its configuration.

---

# Production Note

Many applications need a restart or explicit configuration reload even when mounted files change.

Some applications support hot reload.

Others require

```
Rolling Restart
```

---

# Restart Deployment

```bash
kubectl rollout restart deployment inventory
```

---

# 16. Immutable ConfigMaps

Normally

ConfigMaps

↓

Can Change

---

Sometimes

Production requires

```
Read Only Configuration
```

Example

```yaml
immutable: true
```

---

## Advantages

✔ Better Performance

✔ Prevents Accidental Changes

✔ Safer Production

---

## Example

```yaml
apiVersion: v1

kind: ConfigMap

metadata:

  name: app-config

immutable: true

data:

  APP_NAME: Inventory
```

---

# 17. Common Mistakes

❌ Putting passwords inside ConfigMaps.

❌ Forgetting to create the ConfigMap before deploying the Pod.

❌ Wrong ConfigMap name.

❌ Wrong key name.

❌ Expecting environment variables to update automatically.

❌ Storing large binary files.

---

# 18. Production Best Practices

✔ Keep ConfigMaps environment-specific.

Example

```
inventory-dev

inventory-test

inventory-prod
```

---

✔ Store YAML files in Git.

---

✔ Use immutable ConfigMaps where appropriate.

---

✔ Keep Secrets separate from ConfigMaps.

---

✔ Use meaningful names.

Example

```
payment-config

inventory-config

auth-config
```

---

✔ Version important configuration changes through your deployment process.

---

# 19. Real Production Example

Suppose you have an e-commerce application.

Configuration

```
APP_NAME

↓

Shopping

PORT

↓

8080

LOG_LEVEL

↓

INFO

REDIS_HOST

↓

redis-service
```

Developers build the container image once.

Different environments provide different ConfigMaps.

```
Development

↓

Dev ConfigMap

-------------------

Testing

↓

Test ConfigMap

-------------------

Production

↓

Prod ConfigMap
```

The same application image runs everywhere.

Only configuration changes.

---

# 20. Useful Commands

List ConfigMaps

```bash
kubectl get configmaps
```

Describe

```bash
kubectl describe configmap app-config
```

YAML

```bash
kubectl get configmap app-config -o yaml
```

Edit

```bash
kubectl edit configmap app-config
```

Delete

```bash
kubectl delete configmap app-config
```

---

# 21. Part 2 Summary

In this section, you learned:

- Using ConfigMaps with `env`
- Using `envFrom`
- Mounting ConfigMaps as volumes
- Update behavior
- Immutable ConfigMaps
- Production best practices
- Common mistakes
- Real-world usage

You now understand the three primary ways applications consume ConfigMaps and when to use each approach.

---

# 22. ConfigMap Troubleshooting

Configuration problems are common in Kubernetes.

A structured troubleshooting process helps identify issues quickly.

---

## Step 1 - Verify the ConfigMap Exists

```bash
kubectl get configmaps
```

Expected output:

```
NAME              DATA   AGE
app-config        3      15m
```

If it does not exist:

- Check the namespace.
- Verify the ConfigMap was created successfully.
- Apply the YAML again if necessary.

---

## Step 2 - Describe the ConfigMap

```bash
kubectl describe configmap app-config
```

Check:

- Name
- Namespace
- Keys
- Values
- Events

---

## Step 3 - Verify Namespace

Many "missing ConfigMap" errors happen because the Pod and ConfigMap are in different namespaces.

Check:

```bash
kubectl get configmaps -A
```

---

## Step 4 - Verify Pod Events

```bash
kubectl describe pod inventory
```

Possible error:

```
ConfigMap not found
```

or

```
Couldn't find key APP_PORT
```

These messages usually identify the root cause.

---

## Step 5 - Verify Environment Variables

```bash
kubectl exec -it inventory -- env
```

Check whether expected variables exist.

Example:

```
APP_NAME=Inventory
APP_PORT=8080
```

---

## Step 6 - Verify Mounted Files

```bash
kubectl exec -it inventory -- ls /etc/config
```

Read file contents:

```bash
cat /etc/config/APP_NAME
```

---

## Step 7 - Restart Deployment (If Needed)

When using ConfigMaps as environment variables:

```bash
kubectl rollout restart deployment inventory
```

This recreates Pods so they read the updated values.

---

# Common Problems and Solutions

| Problem | Possible Cause | Solution |
|---------|----------------|----------|
| ConfigMap not found | Wrong namespace or name | Verify namespace and resource name |
| Missing environment variable | Wrong key | Check `configMapKeyRef` |
| Application still using old value | Environment variables don't auto-refresh | Restart the Pod or Deployment |
| Mounted file missing | Incorrect mount path | Verify `volumeMounts` |
| Binary file stored | ConfigMap intended for text | Use an appropriate storage mechanism |

---

# 23. Enterprise Configuration Strategy

Large organizations usually separate configuration by environment.

```
Git Repository

│

├── dev

│     └── configmap.yaml

│

├── test

│     └── configmap.yaml

│

├── stage

│     └── configmap.yaml

│

└── prod

      └── configmap.yaml
```

Each environment has its own configuration while using the same application image.

---

# Example

Development

```yaml
LOG_LEVEL: DEBUG
```

Testing

```yaml
LOG_LEVEL: INFO
```

Production

```yaml
LOG_LEVEL: WARN
```

The container image remains unchanged.

Only the ConfigMap differs.

---

# 24. ConfigMaps in GitOps

GitOps tools such as **Argo CD** and **Flux** treat Git as the source of truth.

Typical workflow:

```
Developer

↓

Git Commit

↓

Git Repository

↓

GitOps Controller

↓

Kubernetes Cluster
```

Benefits:

- Version history
- Rollback support
- Auditable changes
- Automated synchronization

Avoid manually editing production ConfigMaps if Git is your source of truth, because those changes may be overwritten by the GitOps controller.

---

# 25. Kustomize ConfigMap Generator

Kustomize can generate ConfigMaps automatically from files.

Example `kustomization.yaml`:

```yaml
configMapGenerator:
  - name: app-config
    files:
      - app.properties
```

Generate resources:

```bash
kubectl apply -k .
```

### Why Use It?

- Reduces manual YAML maintenance
- Generates names based on content
- Helps trigger rolling updates when configuration changes

---

# 26. Production Best Practices

### Keep ConfigMaps Small

Split unrelated configuration into separate ConfigMaps.

Example:

```
inventory-config

payment-config

notification-config
```

---

### One Responsibility Per ConfigMap

Avoid mixing configuration for multiple applications.

---

### Never Store Sensitive Data

Examples that should **not** be in ConfigMaps:

- Passwords
- API Tokens
- Private Keys
- Database Credentials

Use Kubernetes **Secrets** instead.

---

### Use Labels

```yaml
metadata:
  labels:
    app: inventory
    environment: production
```

Labels make filtering and automation easier.

---

### Use Version Control

Every ConfigMap should be stored in Git.

This provides:

- Change history
- Code review
- Rollback capability

---

# 27. Real Production Scenario

Suppose an online shopping platform has three microservices:

```
Frontend

↓

Inventory

↓

Payment
```

Each service has its own configuration.

```
frontend-config

inventory-config

payment-config
```

Benefits:

- Independent updates
- Clear ownership
- Easier troubleshooting
- Better scalability

---

# 28. Hands-on Labs

## Lab 1 - Create a ConfigMap

```bash
kubectl create configmap app-config \
  --from-literal=APP_NAME=Inventory \
  --from-literal=APP_PORT=8080
```

Verify:

```bash
kubectl get configmaps
```

---

## Lab 2 - Use Environment Variables

Deploy a Pod that reads `APP_NAME` from the ConfigMap.

Verify:

```bash
kubectl exec -it POD_NAME -- env
```

---

## Lab 3 - Mount a ConfigMap

Mount the ConfigMap at:

```
/etc/config
```

Verify:

```bash
ls /etc/config
cat /etc/config/APP_NAME
```

---

## Lab 4 - Update Configuration

Edit the ConfigMap:

```bash
kubectl edit configmap app-config
```

Observe:

- Mounted files update automatically.
- Environment variables require a Pod restart.

---

## Lab 5 - Immutable ConfigMap

Create:

```yaml
immutable: true
```

Attempt to modify it.

Observe Kubernetes rejecting the update.

---

# 29. CKA Exam Tips

✔ Know how to create ConfigMaps from:

- Literals
- Files
- YAML

✔ Understand:

- `env`
- `envFrom`
- Volume mounts

✔ Practice troubleshooting:

```bash
kubectl describe pod
kubectl describe configmap
kubectl exec
```

✔ Remember:

Mounted ConfigMaps and environment variables behave differently when configuration changes.

---

# 30. Interview Questions

## Beginner

1. What is a ConfigMap?
2. Why do we use ConfigMaps?
3. What kind of data belongs in a ConfigMap?
4. Difference between ConfigMap and Secret?
5. How do you create a ConfigMap?

---

## Intermediate

1. Explain `env` vs `envFrom`.
2. How are ConfigMaps mounted as volumes?
3. Why externalize configuration?
4. What is an immutable ConfigMap?
5. What happens when a ConfigMap changes?

---

## Advanced

1. How would you manage ConfigMaps across multiple environments?
2. Explain ConfigMaps in a GitOps workflow.
3. How does Kustomize help manage ConfigMaps?
4. How would you safely roll out a configuration change?
5. How would you troubleshoot a Pod failing due to configuration?

---

## Scenario-Based

### Scenario 1

A Pod reports:

```
ConfigMap not found
```

What checks would you perform?

---

### Scenario 2

A ConfigMap was updated, but the application still uses the old value.

Why?

---

### Scenario 3

A mounted configuration file changed, but the application behavior did not.

What could be the reason?

---

### Scenario 4

You manage separate Dev, Test, and Production environments.

How would you organize ConfigMaps?

---

### Scenario 5

Your organization uses Argo CD.

Should engineers manually edit ConfigMaps in the cluster? Why or why not?

---

# 31. ConfigMap Cheat Sheet

Create from literals:

```bash
kubectl create configmap app-config \
  --from-literal=KEY=value
```

Create from file:

```bash
kubectl create configmap app-config \
  --from-file=app.properties
```

List:

```bash
kubectl get configmaps
```

Describe:

```bash
kubectl describe configmap app-config
```

YAML:

```bash
kubectl get configmap app-config -o yaml
```

Edit:

```bash
kubectl edit configmap app-config
```

Delete:

```bash
kubectl delete configmap app-config
```

Restart Deployment:

```bash
kubectl rollout restart deployment inventory
```

---

# 32. Key Takeaways

- ConfigMaps store **non-sensitive configuration**.
- Configuration should be separated from application code.
- ConfigMaps can be consumed through:
  - Environment variables
  - `envFrom`
  - Volume mounts
- Environment variables typically require Pod recreation after updates.
- Mounted ConfigMap files are updated automatically, but applications may need to reload the configuration.
- Keep ConfigMaps in version control.
- Use separate ConfigMaps for different applications and environments.
- Never store secrets in ConfigMaps.

---

# 33. Chapter Summary

Congratulations! 🎉

You have completed **K8S-08 – ConfigMaps**.

In this chapter you learned:

- What configuration is
- Why ConfigMaps exist
- Problems with hardcoded values
- ConfigMap architecture
- Creating ConfigMaps
- File-based ConfigMaps
- YAML structure
- Environment variables
- `envFrom`
- Volume mounts
- Update behavior
- Immutable ConfigMaps
- Troubleshooting
- GitOps practices
- Kustomize ConfigMap generation
- Production best practices
- Hands-on labs
- CKA tips
- Interview questions

You now have a strong understanding of managing application configuration in Kubernetes. In the next chapter, you'll learn how Kubernetes securely manages sensitive information using **Secrets**, including encryption, secret types, external secret managers, and production security practices.

