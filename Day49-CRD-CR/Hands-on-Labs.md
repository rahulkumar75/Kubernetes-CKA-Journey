# CRD & CR Hands-on Labs and Practice

This practice roadmap is structured from:

* beginner
  → intermediate
  → real-world operator thinking

All labs work well for:

* CKA understanding
* Kubernetes internals
* DevOps interviews
* Operator fundamentals

---

# Lab 1 — Create Your First CRD

## Goal

Understand:

* CRD creation
* API extension
* custom resource registration

---

# Step 1 — Create CRD

## `crd.yaml`

```yaml id="xjlwmk"
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

  scope: Namespaced

  versions:
    - name: v1
      served: true
      storage: true

      schema:
        openAPIV3Schema:
          type: object

          properties:
            spec:
              type: object

              properties:
                engine:
                  type: string

                version:
                  type: string
```

---

# Apply CRD

```bash id="v3p0tq"
kubectl apply -f crd.yaml
```

---

# Verify

```bash id="jjlwm4"
kubectl get crd
```

---

# Check New API Resource

```bash id="jlwm0x"
kubectl api-resources | grep database
```

Expected:

```text id="jlwm3v"
databases   db   mycompany.com/v1
```

---

# Lab 2 — Create First Custom Resource

---

# Step 1 — Create CR

## `db.yaml`

```yaml id="0o7k9q"
apiVersion: mycompany.com/v1
kind: Database

metadata:
  name: mysql-db

spec:
  engine: mysql
  version: "8.0"
```

---

# Apply

```bash id="zwjlwm"
kubectl apply -f db.yaml
```

---

# Verify

```bash id="l5j5kl"
kubectl get databases
```

---

# Describe Resource

```bash id="u4tztx"
kubectl describe database mysql-db
```

---

# Learning Outcome

Understand:

* CRD defines type
* CR creates object

---

# Lab 3 — Validation Schema Practice

## Goal

Learn schema validation.

---

# Update CRD

Add:

```yaml id="jlwmvz"
required:
  - engine

properties:
  engine:
    type: string

    enum:
      - mysql
      - postgres
```

---

# Apply Updated CRD

```bash id="jlwm6p"
kubectl apply -f crd.yaml
```

---

# Test Invalid CR

```yaml id="zv6ngn"
spec:
  engine: mongodb
```

---

# Expected Result

Rejected by API server.

---

# Learning Outcome

Understand:

* OpenAPI validation
* enum validation
* schema enforcement

---

# Lab 4 — Namespaced vs Cluster Scope

---

# Change

```yaml id="jlwm1f"
scope: Cluster
```

---

# Reapply CRD

Delete old CRD first:

```bash id="yjlwmn"
kubectl delete crd databases.mycompany.com
kubectl apply -f crd.yaml
```

---

# Create CR

Now resource becomes:

* cluster-wide

Test:

```bash id="jlwmk4"
kubectl get databases
```

without namespace.

---

# Learning Outcome

Understand:

* resource scoping
* namespace behavior

---

# Lab 5 — Multiple Versions

## Goal

Learn CRD versioning.

---

# Add Versions

```yaml id="jlwmn0"
versions:
  - name: v1alpha1
    served: true
    storage: false

  - name: v1
    served: true
    storage: true
```

---

# Create CRs in Different Versions

---

## Alpha

```yaml id="rj8l5x"
apiVersion: mycompany.com/v1alpha1
kind: Database
```

---

## Stable

```yaml id="u4n1lo"
apiVersion: mycompany.com/v1
kind: Database
```

---

# Learning Outcome

Understand:

* version coexistence
* served/storage concepts

---

# Lab 6 — Additional Printer Columns

## Goal

Improve kubectl output.

---

# Add

```yaml id="c1o0xq"
additionalPrinterColumns:
  - name: Engine
    type: string
    jsonPath: .spec.engine
```

---

# Result

```bash id="jlwmh7"
kubectl get databases
```

shows:

```text id="jlwmrh"
NAME       ENGINE
mysql-db   mysql
```

---

# Learning Outcome

Understand:

* kubectl customization
* CR usability improvement

---

# Lab 7 — Status Subresource

## Goal

Understand desired vs actual state.

---

# Add to CRD

```yaml id="jlwm8t"
subresources:
  status: {}
```

---

# Create CR

Then manually patch status:

```bash id="jlwm7y"
kubectl patch database mysql-db \
  --type merge \
  -p '{"status":{"phase":"Running"}}'
```

---

# Verify

```bash id="jlwm2g"
kubectl get database mysql-db -o yaml
```

---

# Learning Outcome

Understand:

* spec = desired state
* status = current state

---

# Lab 8 — Watch API Events

## Goal

Observe reconciliation-style behavior.

---

# Terminal 1

```bash id="jjlwm6"
kubectl get databases -w
```

---

# Terminal 2

Create/update/delete CRs.

Observe:

* watch stream
* real-time events

---

# Learning Outcome

Understand:

* watch mechanism
* controller foundations

---

# Lab 9 — Simulate Operator Thinking

## Goal

Think like a controller.

---

# Create CR

```yaml id="jlwm0q"
spec:
  replicas: 3
```

---

# Imagine Controller Logic

```text id="jlwmm4"
Desired replicas = 3
Actual replicas = 1
↓
Controller creates 2 more pods
```

---

# Learning Outcome

Understand:

* reconciliation loop
* operator mindset

---

# Lab 10 — Explore Real CRDs

---

# Install Cert-Manager

Then:

```bash id="0jlwm1"
kubectl get crd
```

Observe:

* Certificate
* Issuer
* ClusterIssuer

---

# Inspect Real CRD

```bash id="jlwm93"
kubectl get crd certificates.cert-manager.io -o yaml
```

---

# Learning Outcome

Understand:

* production-grade CRDs
* enterprise schema design

---

# Advanced Practice (Optional)

| Practice            | Purpose             |
| ------------------- | ------------------- |
| Kubebuilder         | Build operators     |
| Operator SDK        | Create controllers  |
| Conversion webhooks | Version conversion  |
| Admission webhooks  | Advanced validation |
| Crossplane CRDs     | Infrastructure APIs |

---

# Important Practice Checklist

| Topic                | Done |
| -------------------- | ---- |
| Create CRD           | ✅    |
| Create CR            | ✅    |
| Validation schema    | ✅    |
| Versioning           | ✅    |
| Namespaced/Cluster   | ✅    |
| Printer columns      | ✅    |
| Status field         | ✅    |
| Watch API            | ✅    |
| Real CRDs inspection | ✅    |

---

# Best Practice for Learning

## Focus on Understanding:

```text id="jlwm6w"
CRD → API Extension
CR → Desired State
Controller → Reconciliation
Status → Actual State
```

This is the core mental model.

---

# One-Line Interview Summary

> Hands-on CRD practice should include creating custom APIs, validation schemas, versioning, status handling, and understanding how controllers reconcile Custom Resources into actual system state.
