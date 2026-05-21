## Kubernetes **Custom Resource Definitions (CRDs)** and **Custom Resources (CRs)**.

Topics can include:

* What CRD and CR are
* Architecture & flow
* YAML structure
* API extensions
* Operators & Controllers
* Versioning (`v1alpha1`, `v1beta1`, `v1`)
* Validation schemas
* Subresources (`status`, `scale`)
* Conversion webhooks
* Kubebuilder / Operator SDK
* Real-world examples
* Interview questions
* Troubleshooting
* Hands-on labs and practice

We’ll keep everything centered around CRDs and CRs only.


## CRD and CR in Kubernetes

### What is a CRD (Custom Resource Definition)?

A **CRD** is a way to extend the Kubernetes API.

It allows you to create your own Kubernetes object type, just like:

* Pod
* Deployment
* Service

Example custom types:

* `Database`
* `Backup`
* `AppConfig`
* `MySQLCluster`

CRD defines:

* resource name
* API group
* versions
* schema/validation
* object structure

---

### What is a CR (Custom Resource)?

A **CR** is an actual instance/object created from a CRD.

Just like:

* Deployment YAML creates a Deployment object

Similarly:

* Custom YAML creates a Custom Resource object

---

# Flow

```text
CRD Created
↓
Kubernetes API learns new resource type
↓
User creates CR object
↓
Controller/Operator watches CR
↓
Controller performs actual work
```

---

# Real Example

## Step 1 — Create CRD

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

## Step 2 — Create CR

```yaml
apiVersion: mycompany.com/v1
kind: Database

metadata:
  name: mysql-db

spec:
  engine: mysql
  version: "8.0"
```

---

# What Happens Internally?

After CRD creation:

```bash
kubectl api-resources
```

You will see:

```text
databases   db   mycompany.com/v1
```

Now Kubernetes understands a new object type called:

```text
Database
```

Then users can create CR objects.

---

# Important Difference

| CRD                       | CR                     |
| ------------------------- | ---------------------- |
| Defines new resource type | Actual object instance |
| Like class/template       | Like object/instance   |
| Extends Kubernetes API    | Uses extended API      |
| Created once              | Can create many        |

---

# Key Commands

## Create CRD

```bash
kubectl apply -f crd.yaml
```

## Create CR

```bash
kubectl apply -f cr.yaml
```

## Get custom resources

```bash
kubectl get databases
```

## Explain resource

```bash
kubectl explain database
```

---

# Why CRD is Powerful?

CRDs make Kubernetes extensible.

Many popular tools use CRDs internally:

* Argo CD
* Cert-Manager
* Prometheus Operator
* Istio

Example:

* `Certificate`
* `Application`
* `ServiceMonitor`
* `VirtualService`

All are custom resources.

---

# Simple Analogy

| Real World                 | Kubernetes          |
| -------------------------- | ------------------- |
| Form template              | CRD                 |
| Filled form                | CR                  |
| School management software | Controller/Operator |

---

# One-Line Interview Answer

> A CRD extends the Kubernetes API by defining a new custom resource type, while a CR is an actual instance of that custom resource created by users or controllers.
