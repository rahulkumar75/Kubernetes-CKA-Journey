# CRD Versioning in Kubernetes

CRD versioning allows a custom resource API to evolve safely over time.

Example versions:

* `v1alpha1`
* `v1beta1`
* `v1`

---

# Why Versioning is Needed?

Suppose your first CRD looks like:

```yaml id="j6l5x9"
spec:
  engine: mysql
```

Later you want:

```yaml id="n2e6q0"
spec:
  engine: mysql
  storage: 20Gi
  replicas: 3
```

Changing schema directly may:

* break old clients
* break controllers
* break automation

So Kubernetes supports:

* multiple API versions.

---

# Common Version Stages

| Version  | Meaning                | Stability     |
| -------- | ---------------------- | ------------- |
| v1alpha1 | Early development      | Unstable      |
| v1beta1  | Testing/pre-production | Mostly stable |
| v1       | Production-ready       | Stable        |

---

# 1. v1alpha1

Earliest stage.

Characteristics:

* experimental
* breaking changes possible
* not production safe
* schema changes frequent

Used for:

* development
* testing
* internal experiments

---

# 2. v1beta1

More mature.

Characteristics:

* mostly stable
* APIs may still change
* production testing phase

Used for:

* wider adoption
* pre-production workloads

---

# 3. v1

Stable release.

Characteristics:

* backward compatibility expected
* production ready
* schema stable
* safest version

---

# CRD Version Structure

Example:

```yaml id="0c6r6h"
versions:
  - name: v1alpha1
    served: true
    storage: false

  - name: v1beta1
    served: true
    storage: false

  - name: v1
    served: true
    storage: true
```

---

# Important Fields

---

# served

```yaml id="6bgv6y"
served: true
```

Means:

* this API version is accessible.

Users can use:

```yaml id="nzw4w2"
apiVersion: mycompany.com/v1beta1
```

---

# storage

```yaml id="wyi07p"
storage: true
```

Defines:

* which version is stored in etcd.

Only ONE version can have:

```yaml id="kpmmbo"
storage: true
```

---

# Example Flow

```text id="l7pofx"
User creates CR in v1beta1
        ↓
API server converts object
        ↓
Stored internally as storage version
        ↓
Returned back in requested version
```

---

# Multi-Version Example

## CRD

```yaml id="uvg86w"
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition

metadata:
  name: databases.mycompany.com

spec:
  group: mycompany.com

  names:
    plural: databases
    singular: database
    kind: Database

  scope: Namespaced

  versions:
    - name: v1alpha1
      served: true
      storage: false

      schema:
        openAPIV3Schema:
          type: object

    - name: v1
      served: true
      storage: true

      schema:
        openAPIV3Schema:
          type: object
```

---

# Using Different Versions

---

## Create CR using alpha version

```yaml id="3oq5k9"
apiVersion: mycompany.com/v1alpha1
kind: Database
```

---

## Create CR using stable version

```yaml id="l95mbm"
apiVersion: mycompany.com/v1
kind: Database
```

---

# Conversion Between Versions

Sometimes schemas differ between versions.

Example:

---

## v1alpha1

```yaml id="zztv1j"
spec:
  size: small
```

---

## v1

```yaml id="4n6g1r"
spec:
  storage: 20Gi
```

Kubernetes may need:

* conversion logic.

---

# Conversion Strategies

| Strategy | Meaning                 |
| -------- | ----------------------- |
| None     | No conversion           |
| Webhook  | Custom conversion logic |

---

# Conversion Webhook

Advanced mechanism.

Used when:

* field names change
* schema changes heavily
* automatic conversion impossible

---

# Architecture

```text id="3j8o0u"
Request v1alpha1
       ↓
API Server
       ↓
Conversion Webhook
       ↓
Converted to v1
       ↓
Stored in etcd
```

---

# Version Migration Practice

Typical lifecycle:

```text id="8gv0o4"
v1alpha1
    ↓
v1beta1
    ↓
v1
```

Older versions may later become:

```yaml id="vjlwmj"
served: false
```

meaning:

* users can no longer access them.

---

# Deprecation Example

```yaml id="s3n9ih"
- name: v1alpha1
  served: false
  storage: false
```

---

# Important Interview Concepts

---

# 1. Multiple Versions Can Exist Together

Users may use:

* v1alpha1
* v1beta1
* v1

simultaneously.

---

# 2. Only One Storage Version

Exactly one:

```yaml id="vx16f7"
storage: true
```

---

# 3. Versioning Enables Safe API Evolution

Without breaking existing workloads.

---

# Real-World Examples

| Tool         | Versions              |
| ------------ | --------------------- |
| Istio        | v1alpha3, v1beta1, v1 |
| Cert-Manager | v1alpha2 → v1         |
| Argo CD      | evolving CRD APIs     |

---

# Common Commands

## Check CRD versions

```bash id="g5n0m3"
kubectl get crd databases.mycompany.com -o yaml
```

---

## Check API versions

```bash id="4oj9u2"
kubectl api-versions
```

---

# Important Best Practice

| Recommendation                            | Reason           |
| ----------------------------------------- | ---------------- |
| Use v1 for production                     | Stable           |
| Keep backward compatibility               | Prevent breakage |
| Use conversion webhook for schema changes | Safe migrations  |
| Avoid long-term alpha APIs                | Unstable         |

---

# One-Line Interview Answer

> CRD versioning allows Kubernetes custom APIs to evolve safely using versions like v1alpha1, v1beta1, and v1, where multiple API versions can coexist while one storage version is maintained internally in etcd.


# Error Scenario

The error happens because in `apiextensions.k8s.io/v1`, **every served version must have a schema**.

In your YAML:

```yaml
- name: v1alpha1
  served: true
  storage: false
```

`v1alpha1` is `served: true`, but it does not contain:

```yaml
schema:
  openAPIV3Schema:
```

That’s why Kubernetes says:

```bash
spec.versions[0].schema.openAPIV3Schema: Required value
```

---

# Fix Options

## Option 1 (Recommended)

Add schema to both versions.

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition

metadata:
  name: databases.mycompany.com

spec:
  group: mycompany.com

  names:
    plural: databases
    singular: database
    kind: Database
    shortNames:
      - db

  scope: Cluster

  versions:
    - name: v1alpha1
      served: true
      storage: false

      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              required:
                - engine
              properties:
                engine:
                  type: string
                  enum:
                    - mysql
                    - postgres
                version:
                  type: string

    - name: v1
      served: true
      storage: true

      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              required:
                - engine
              properties:
                engine:
                  type: string
                  enum:
                    - mysql
                    - postgres
                version:
                  type: string
```

---

# Option 2

If you don’t want `v1alpha1` right now:

```yaml
- name: v1alpha1
  served: false
  storage: false
```

Then schema is not required.

---

# Important Interview Point

In CRDs:

* `served: true`
  → API is accessible to users.

* `storage: true`
  → Version stored in etcd.

Only **one** version can have:

```yaml
storage: true
```

---

# Real-world Versioning Flow

Typical evolution:

```text
v1alpha1  → experimental
v1beta1   → stable testing
v1        → production stable
```

Example:

* `v1alpha1` → basic DB fields
* `v1beta1` → backup support
* `v1` → HA + replication + validations

---

# Quick Validation Test

After applying:

```bash
kubectl get crd
```

Then:

```bash
kubectl api-resources | grep database
```

Expected:

```bash
databases   db   mycompany.com/v1   true   Database
```

You can also inspect versions:

```bash
kubectl get crd databases.mycompany.com -o yaml
```
---

Your CRD is applied correctly, but the issue is most likely because:

```yaml
scope: Cluster
```

Your CRD is **Cluster-scoped**, but your Custom Resource (CR) contains:

```yaml
namespace: db
```

Cluster-scoped resources do NOT belong to namespaces.

So Kubernetes ignores namespace behavior, and sometimes users think the resource/version is not working.

---

# Fix

Remove namespace:

```yaml
apiVersion: mycompany.com/v1alpha1
kind: Database

metadata:
  name: mysql-db-alpha

spec:
  engine: postgres
  version: "8.0"
```

Apply again:

```bash
kubectl apply -f cr.yaml
```

---

# Then Verify

Check all databases:

```bash
kubectl get databases
```

or:

```bash
kubectl get db
```

---

# Verify Version: 

Another Important Thing


Your CRD has:

```yaml
- name: v1alpha1
  served: true
  storage: false

- name: v1
  served: true
  storage: true
```

Meaning:

| Version  | Purpose                   |
| -------- | ------------------------- |
| v1alpha1 | API exposed to users      |
| v1       | Stored internally in etcd |

So even if you create:

```yaml
apiVersion: mycompany.com/v1alpha1
```

Kubernetes may internally convert/store it as `v1`.

This is normal behavior.

---

# Best Command for Debugging Versions

```bash
kubectl describe crd databases.mycompany.com
```

Look for:

```text
Versions:
  v1alpha1
    Served: true
    Storage: false

  v1
    Served: true
    Storage: true
```

---

# Interview-Level Understanding

## `served: true`

Clients can use this API version.

Example:

```bash
kubectl get databases.v1alpha1.mycompany.com
```

works.

---

## `storage: true`

Actual persisted version in etcd.

Only one version can be storage version.

---

# Real-world Flow

Users may use:

```text
v1alpha1
```

But controllers/operators usually convert everything internally to:

```text
v1
```

This is called:

* Version conversion
* API evolution
* Backward compatibility

Used heavily by:

* Operators
* Istio
* Cert Manager
* ArgoCD
* Crossplane
