# CRD YAML Structure

A CRD YAML mainly contains:

```text id="jz5a1r"
apiVersion
kind
metadata
spec
```

---

# Full CRD YAML Example

```yaml id="e70n1q"
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

                storage:
                  type: string
```

---

# Structure Breakdown

---

# 1. apiVersion

```yaml id="pl0n42"
apiVersion: apiextensions.k8s.io/v1
```

Defines:

* which Kubernetes API is used for CRD creation.

CRDs are managed by:

* `apiextensions.k8s.io`

---

# 2. kind

```yaml id="7yegbx"
kind: CustomResourceDefinition
```

Tells Kubernetes:

* this object is a CRD.

---

# 3. metadata

```yaml id="k9wd86"
metadata:
  name: databases.mycompany.com
```

Format:

```text id="9gkjm3"
<plural>.<group>
```

Example:

* `databases.mycompany.com`

---

# 4. spec

Main configuration area.

---

# 5. group

```yaml id="0ndh7k"
group: mycompany.com
```

Creates API group:

```text id="gnj5d0"
mycompany.com
```

Final API becomes:

```text id="f19iqx"
mycompany.com/v1
```

---

# 6. names

Defines resource naming.

```yaml id="o26a4w"
names:
  plural: databases
  singular: database
  kind: Database
  shortNames:
    - db
```

---

## plural

Used in API URLs:

```text id="88q0i4"
/apis/mycompany.com/v1/databases
```

---

## singular

Used internally.

---

## kind

Object type name.

Example:

```yaml id="knc9ot"
kind: Database
```

---

## shortNames

Shortcut command:

```bash id="nh5z4f"
kubectl get db
```

---

# 7. scope

```yaml id="4mnh1y"
scope: Namespaced
```

Possible values:

| Value      | Meaning                 |
| ---------- | ----------------------- |
| Namespaced | Exists inside namespace |
| Cluster    | Cluster-wide resource   |

---

## Example

### Namespaced

```bash id="51t0o8"
kubectl get databases -n dev
```

### Cluster

```bash id="nsm48p"
kubectl get nodes
```

(no namespace)

---

# 8. versions

Defines API versions.

```yaml id="vokny0"
versions:
  - name: v1
```

Supports:

* v1alpha1
* v1beta1
* v1

---

# Important Fields

---

## served

```yaml id="ok2v6x"
served: true
```

Means:

* API version accessible to users.

---

## storage

```yaml id="r7qk9l"
storage: true
```

Defines:

* which version stored in etcd.

Only ONE storage version allowed.

---

# 9. schema

Defines validation rules.

```yaml id="7a0xgg"
schema:
  openAPIV3Schema:
```

Kubernetes validates CR objects before accepting them.

---

# 10. properties

Defines allowed fields.

```yaml id="7lmq8c"
properties:
  spec:
```

---

# Example Validation

```yaml id="1y31lt"
engine:
  type: string
```

If user gives:

```yaml id="dh5sk8"
engine: 123
```

Kubernetes rejects it.

---

# CR YAML Structure

After CRD creation, users create CR objects.

---

# Example CR

```yaml id="d7uw1p"
apiVersion: mycompany.com/v1
kind: Database

metadata:
  name: mysql-db

spec:
  engine: mysql
  version: "8.0"
  storage: 20Gi
```

---

# CR Structure Breakdown

| Field      | Purpose                  |
| ---------- | ------------------------ |
| apiVersion | Custom API group/version |
| kind       | CR type                  |
| metadata   | Object info              |
| spec       | Desired state            |

---

# Important Relationship

```text id="qifshl"
CRD defines structure
CR follows structure
```

---

# Architecture Mapping

| CRD Section | Purpose           |
| ----------- | ----------------- |
| group       | API group         |
| versions    | API version       |
| names       | Resource naming   |
| schema      | Validation        |
| scope       | Namespace/Cluster |
| spec in CR  | Desired state     |

---

# Very Important Interview Point

## CRD adds API endpoint dynamically

Example:

```text id="qmxzcn"
/apis/mycompany.com/v1/databases
```

without modifying Kubernetes source code.

---

# Common kubectl Commands

## Check CRD

```bash id="az5yga"
kubectl get crd
```

---

## Describe CRD

```bash id="dvy0j0"
kubectl describe crd databases.mycompany.com
```

---

## Check custom resources

```bash id="shz2kj"
kubectl get databases
```

---

# One-Line Interview Answer

> A CRD YAML defines a new Kubernetes API resource using fields like group, versions, names, scope, and schema, while a CR YAML creates an actual object that follows the CRD structure.
