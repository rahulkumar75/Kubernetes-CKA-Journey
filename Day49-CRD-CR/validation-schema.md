# Validation Schemas in CRDs

Validation schemas define:

* what fields are allowed
* field data types
* required fields
* validation rules

for Custom Resources (CRs).

Kubernetes validates CR objects before storing them in etcd.

---

# Why Validation is Important?

Without validation:

```yaml id="z3g99x"
spec:
  engine: 123
```

or:

```yaml id="vmvvpi"
spec:
  storage: abcxyz
```

could be accepted.

This may break:

* controllers
* operators
* automation logic

Validation schemas prevent invalid data.

---

# Validation Uses OpenAPI v3 Schema

CRDs use:

```yaml id="mqjqm0"
openAPIV3Schema
```

---

# Basic Structure

```yaml id="3h1z80"
schema:
  openAPIV3Schema:
```

---

# Full Example

```yaml id="jndmta"
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
                - version

              properties:
                engine:
                  type: string

                version:
                  type: string

                replicas:
                  type: integer

                storage:
                  type: string
```

---

# Structure Breakdown

---

# type

Defines data type.

---

## String

```yaml id="84mp0o"
type: string
```

Example:

```yaml id="i2u4eq"
engine: mysql
```

---

## Integer

```yaml id="r2s4ot"
type: integer
```

Example:

```yaml id="rx4i4d"
replicas: 3
```

---

## Boolean

```yaml id="0w5vcd"
type: boolean
```

Example:

```yaml id="6wjffq"
enabled: true
```

---

## Array

```yaml id="8pk6mx"
type: array
```

---

## Object

```yaml id="7axt5h"
type: object
```

Used for nested structures.

---

# properties

Defines allowed fields.

```yaml id="xgtx62"
properties:
```

---

# required

Makes fields mandatory.

```yaml id="lt0qlc"
required:
  - engine
  - version
```

If missing:

```yaml id="4h1t5v"
spec:
  engine: mysql
```

Kubernetes rejects the CR.

---

# Example CR (Valid)

```yaml id="e5r2jd"
apiVersion: mycompany.com/v1
kind: Database

metadata:
  name: prod-db

spec:
  engine: mysql
  version: "8.0"
  replicas: 3
```

Accepted successfully.

---

# Example CR (Invalid)

```yaml id="7mnr2d"
spec:
  engine: 123
```

Error:

```text id="njtkk3"
expected string, got integer
```

---

# Nested Validation

Example:

```yaml id="fh2v6s"
properties:
  spec:
    type: object

    properties:
      backup:
        type: object

        properties:
          enabled:
            type: boolean

          schedule:
            type: string
```

---

# Enum Validation

Restricts allowed values.

```yaml id="awx9mp"
engine:
  type: string

  enum:
    - mysql
    - postgres
```

---

# Valid

```yaml id="rb1qiw"
engine: mysql
```

---

# Invalid

```yaml id="x14kzj"
engine: mongodb
```

Rejected.

---

# String Pattern Validation

Regex validation.

```yaml id="ec72b0"
version:
  type: string
  pattern: "^[0-9]+\\.[0-9]+$"
```

Accepts:

* `8.0`
* `15.2`

Rejects:

* `latest`

---

# Number Range Validation

---

## Minimum

```yaml id="6m3l44"
replicas:
  type: integer
  minimum: 1
```

---

## Maximum

```yaml id="d1n88j"
maximum: 10
```

---

# Default Values

```yaml id="mnl7pr"
replicas:
  type: integer
  default: 1
```

If user omits replicas:

* Kubernetes automatically sets:

  * `replicas: 1`

---

# Nullable Fields

```yaml id="iuhrrm"
nullable: true
```

Allows:

```yaml id="1sjqbh"
storage: null
```

---

# Additional Printer Columns

Enhances:

```bash id="gfl8tx"
kubectl get
```

output.

Example:

```yaml id="jjlwmg"
additionalPrinterColumns:
  - name: Engine
    type: string
    jsonPath: .spec.engine
```

---

# Validation Flow

```text id="yqeblu"
User submits CR
       ↓
API Server validates schema
       ↓
If valid → stored in etcd
If invalid → rejected
```

---

# Important Benefits

| Benefit                | Description       |
| ---------------------- | ----------------- |
| Prevent invalid CRs    | Safer APIs        |
| Protect controllers    | Avoid crashes     |
| Improve consistency    | Standardized data |
| Better user experience | Early validation  |

---

# Important Limitation

Validation schema:

* validates structure/data

BUT does NOT:

* execute business logic
* create infrastructure

Controllers still handle:

* reconciliation
* automation

---

# Real-World Example

## Cert-Manager

Certificate CRDs validate:

* DNS names
* issuer references
* secret names

before certificates are created.

---

# Common kubectl Commands

## Check CRD schema

```bash id="5j6wzq"
kubectl get crd databases.mycompany.com -o yaml
```

---

## Explain CR structure

```bash id="fiv65s"
kubectl explain database.spec
```

---

# Interview-Level Understanding

Validation schemas make CRDs behave like:

* strongly typed APIs

instead of:

* unstructured YAML storage.

---

# One-Line Interview Answer

> Validation schemas in CRDs use OpenAPI v3 schema definitions to validate Custom Resources by enforcing field types, required fields, enums, patterns, and other rules before objects are stored in etcd.
