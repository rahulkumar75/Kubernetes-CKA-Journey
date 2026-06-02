# Kubernetes CRD & CR Mind Map (0–3 YOE Interview Revision)

```text
                         ┌─────────────────────────┐
                         │     CRD & CR in K8s     │
                         └───────────┬─────────────┘
                                     │
      ┌──────────────────────────────┼──────────────────────────────┐
      │                              │                              │
      ▼                              ▼                              ▼

┌───────────────┐          ┌─────────────────┐           ┌─────────────────┐
│      CRD      │          │       CR        │           │   Controller    │
│ API Extension │          │ Desired State   │           │ Reconciliation  │
└───────┬───────┘          └────────┬────────┘           └────────┬────────┘
        │                           │                             │
        │                           │                             │
        ▼                           ▼                             ▼

Defines New API         Instance of CRD               Watches Custom Resources
Resource Type           Created by User               Using Informers

Examples:               Example:                       Compares:
Database                mysql-db                      Desired vs Actual
Application             webapp-prod
Certificate                                            Creates/Updates
                                                      Deployments/Services

Adds Endpoint:
 /apis/group/version/resource

Example:
 /apis/mycompany.com/v1/databases

────────────────────────────────────────────────────────────────────

                    COMPLETE ARCHITECTURE FLOW

      CRD Created
            │
            ▼
 API Server Registers New API
            │
            ▼
 kubectl api-resources
            │
            ▼
 User Creates CR
            │
            ▼
 CR Stored in etcd
            │
            ▼
 Controller Watches CR
            │
            ▼
 Reconciliation Loop
            │
            ▼
 Creates Resources
 (Deployment/Service)
            │
            ▼
 Updates Status
            │
            ▼
 User Views Current State

────────────────────────────────────────────────────────────────────

                 CRD YAML STRUCTURE

apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition

metadata:
  name: databases.mycompany.com

spec:
 ├── group
 ├── names
 │    ├── plural
 │    ├── singular
 │    ├── kind
 │    └── shortNames
 ├── scope
 ├── versions
 └── schema

────────────────────────────────────────────────────────────────────

                  CR YAML STRUCTURE

apiVersion: mycompany.com/v1
kind: Database

metadata:
  name: mysql-db

spec:
  engine: mysql

spec
  │
  ▼
Desired State

status
  │
  ▼
Actual State

────────────────────────────────────────────────────────────────────

                 VERSIONING

v1alpha1
   │
   ▼
Experimental

v1beta1
   │
   ▼
Testing / Mostly Stable

v1
   │
   ▼
Production Ready

Important:
✔ Multiple versions can coexist
✔ Only one storage:true

────────────────────────────────────────────────────────────────────

              VALIDATION SCHEMA

openAPIV3Schema

Supports:

✔ type
✔ required
✔ enum
✔ pattern
✔ minimum
✔ maximum
✔ default

Example:

engine:
  type: string
  enum:
    - mysql
    - postgres

────────────────────────────────────────────────────────────────────

              IMPORTANT COMPONENTS

CRD
 │
 └─ Defines API

CR
 │
 └─ Desired State

etcd
 │
 └─ Stores Objects

Informer
 │
 └─ Watches Changes

Controller
 │
 └─ Reconciliation Logic

Status
 │
 └─ Actual State

────────────────────────────────────────────────────────────────────

                 REAL-WORLD EXAMPLES

Argo CD
 └─ Application CR

Cert-Manager
 └─ Certificate CR

Prometheus Operator
 └─ ServiceMonitor CR

Istio
 └─ VirtualService CR

Crossplane
 └─ EC2 / RDS CRs

────────────────────────────────────────────────────────────────────

                INTERVIEW KEYWORDS

CRD = API Extension

CR = Desired State

Informer = Event Listener

Controller = Automation Brain

Reconciliation =
Desired State == Actual State

Status =
Current Cluster State

Operator =
Controller + Domain Knowledge

etcd =
Storage Backend
```

## 30-Second Interview Answer

> A CRD extends the Kubernetes API by defining a new resource type, while a CR is an instance of that resource representing the desired state. Controllers watch CRs using informers, run reconciliation loops to match actual state with desired state, and update the status field to reflect the current state. This pattern is used by tools like Argo CD, Cert-Manager, and Prometheus Operator.
