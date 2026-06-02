# CRD & CR Interview Questions (0–2 YOE)

---

# Basic Understanding

## 1. What is a CRD in Kubernetes?

> CRD (CustomResourceDefinition) is a Kubernetes API extension mechanism used to create custom resource types dynamically without modifying Kubernetes source code.

---

## 2. What is a Custom Resource (CR)?

> A Custom Resource is an instance/object created from a CRD.

Example:

* CRD defines `Database`
* CR creates `mysql-db`

---

## 3. Difference between CRD and CR?

| CRD                       | CR                     |
| ------------------------- | ---------------------- |
| Defines new resource type | Actual object instance |
| Extends API               | Uses API               |
| Created once              | Multiple can exist     |

---

## 4. Why are CRDs used?

CRDs are used to:

* extend Kubernetes APIs
* build operators
* automate infrastructure/platform tasks
* create domain-specific APIs

---

## 5. What happens after creating a CRD?

```text id="jlwm7x"
CRD created
    ↓
API server registers new endpoint
    ↓
kubectl can access new resource
```

Example:

```bash id="jj27m4"
kubectl get databases
```

---

# Architecture & Flow

## 6. Explain CRD architecture flow.

```text id="fjlwmw"
CRD
 ↓
API endpoint created
 ↓
CR created
 ↓
Stored in etcd
 ↓
Controller watches CR
 ↓
Controller reconciles desired state
```

---

## 7. Does CRD alone perform automation?

> No.

CRD only:

* defines schema/API

Automation is done by:

* controllers/operators

---

## 8. Where are Custom Resources stored?

> In etcd, same as native Kubernetes objects.

---

## 9. What component watches CRs?

> Controllers or Operators watch CRs using Kubernetes Watch API.

---

# YAML & API

## 10. Important sections in CRD YAML?

Main sections:

* group
* versions
* names
* scope
* schema

---

## 11. What is `group` in CRD?

Example:

```yaml id="vjlwm5"
group: mycompany.com
```

Creates API group:

```text id="ycn5hn"
mycompany.com/v1
```

---

## 12. Difference between `kind` and `plural`?

| Field  | Example   |
| ------ | --------- |
| kind   | Database  |
| plural | databases |

---

## 13. Difference between Namespaced and Cluster scope?

| Scope      | Meaning                 |
| ---------- | ----------------------- |
| Namespaced | Exists inside namespace |
| Cluster    | Cluster-wide            |

---

# Validation & Schema

## 14. What is `openAPIV3Schema`?

> Used to validate Custom Resource fields and structure.

---

## 15. Why validation schema is important?

It:

* prevents invalid CRs
* improves API reliability
* protects controllers

---

## 16. What validations are possible?

* type validation
* required fields
* enums
* regex patterns
* min/max values
* defaults

---

## 17. What happens if validation fails?

> API server rejects the CR before storing it in etcd.

---

# Versioning

## 18. Explain `v1alpha1`, `v1beta1`, and `v1`.

| Version  | Meaning            |
| -------- | ------------------ |
| v1alpha1 | Experimental       |
| v1beta1  | Testing/stable-ish |
| v1       | Production stable  |

---

## 19. Can multiple CRD versions exist together?

> Yes.

Example:

* v1alpha1
* v1beta1
* v1

can coexist.

---

## 20. What does `served: true` mean?

> API version accessible to users.

---

## 21. What does `storage: true` mean?

> Defines which version is stored internally in etcd.

Only one version can have:

* `storage: true`

---

# Controller & Operator

## 22. What is reconciliation loop?

Controller continuously compares:

* desired state
* actual state

and fixes differences.

---

## 23. Difference between Controller and Operator?

| Controller       | Operator                   |
| ---------------- | -------------------------- |
| Basic automation | Advanced domain automation |
| Generic          | Application-aware          |

---

## 24. Can CRDs work without Operators?

> Yes, but they become only stored objects with no automation.

---

# Commands

## 25. How to list CRDs?

```bash id="m0d7lj"
kubectl get crd
```

---

## 26. How to check API resources?

```bash id="fjlwmq"
kubectl api-resources
```

---

## 27. How to explain custom resource fields?

```bash id="7lg8gd"
kubectl explain database.spec
```

---

# Real-World Examples

## 28. Name tools using CRDs.

Examples:

* Argo CD
* Cert-Manager
* Istio
* Prometheus Operator

---

## 29. Example CRs from real tools?

| Tool         | CR             |
| ------------ | -------------- |
| Argo CD      | Application    |
| Cert-Manager | Certificate    |
| Istio        | VirtualService |

---

# Scenario-Based Questions

## 30. You created a CR but nothing happened. Why?

Possible reasons:

* controller/operator not installed
* controller crashed
* reconciliation failed
* CR invalid
* RBAC issue

---

## 31. How does Kubernetes know a new resource exists?

> kube-apiserver dynamically registers it after CRD creation.

---

## 32. Can CRDs replace native Kubernetes resources?

> No, but they can extend Kubernetes functionality.

---

# Important Interview Tip

For 0–2 YOE interviews:
focus more on:

* concepts
* flow
* architecture
* real-world usage

instead of:

* deep operator SDK internals
* conversion webhook implementation

---

# Best Short Summary Answer

> CRDs extend Kubernetes APIs by defining new resource types, while controllers/operators watch Custom Resources and reconcile actual state with desired state to automate platform operations.
