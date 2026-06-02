# CRD & CR Architecture and Flow

## High-Level Architecture

```text id="9wd0lr"
                User / kubectl
                       |
                       v
              Kubernetes API Server
                       |
         --------------------------------
         |                              |
         v                              v
   Built-in Resources            Custom Resources
   (Pod, Service)               (Database, App, etc.)
                                         |
                                         v
                         CustomResourceDefinition (CRD)
                                         |
                                         v
                              etcd stores CR objects
                                         |
                                         v
                           Controller / Operator watches
                                         |
                                         v
                              Actual infrastructure work
```

---

# Complete Flow

```text id="l3xy2g"
1. Create CRD
        ↓
2. API Server registers new API endpoint
        ↓
3. Kubernetes understands new resource type
        ↓
4. User creates Custom Resource (CR)
        ↓
5. CR stored in etcd
        ↓
6. Controller/Operator watches CR
        ↓
7. Controller takes action
        ↓
8. Actual state becomes desired state
        ↓
9. Status updated back to CR
```

---

# Step-by-Step Deep Explanation

---

# 1. CRD Creation

Admin applies CRD YAML:

```yaml id="w6u1qg"
kind: CustomResourceDefinition
```

This tells Kubernetes:

> "A new resource type exists."

Example:

* Database
* KafkaCluster
* Application

---

# 2. API Server Registers Resource

Kubernetes API Server dynamically creates endpoints.

Example:

```text id="g2gqtp"
/apis/mycompany.com/v1/databases
```

Now Kubernetes can handle:

```bash id="xgptt2"
kubectl get databases
kubectl create -f db.yaml
```

without changing Kubernetes source code.

---

# 3. Resource Stored in etcd

Custom Resources are stored inside:

* etcd

Same as:

* Pods
* Deployments
* Services

Kubernetes treats CRs as native objects.

---

# 4. User Creates CR

Example:

```yaml id="jlwmkl"
apiVersion: mycompany.com/v1
kind: Database

metadata:
  name: prod-db

spec:
  engine: mysql
  size: large
```

This only declares:

```text id="cnulwo"
Desired State
```

It does NOT create real infrastructure automatically.

---

# 5. Controller / Operator Watches CR

Controller continuously watches:

```text id="xwhm1t"
Database objects
```

using Kubernetes Watch API.

---

# 6. Reconciliation Loop Starts

Controller compares:

| Desired State         | Actual State     |
| --------------------- | ---------------- |
| MySQL DB should exist | DB may not exist |

If mismatch happens:

* controller creates DB
* updates config
* restarts workload
* scales resources

This is called:

```text id="g0a2hk"
Reconciliation Loop
```

---

# 7. Controller Updates Status

Controller writes current state into:

```yaml id="tmpp0d"
status:
  phase: Running
  endpoint: mysql.example.com
```

Now user can check:

```bash id="2e8krg"
kubectl get database
kubectl describe database prod-db
```

---

# Real Kubernetes Flow Example

## Example:

Using Cert-Manager

### User Creates:

```yaml id="18tm91"
kind: Certificate
```

### cert-manager Controller:

* watches Certificate CR
* talks to Let's Encrypt
* generates TLS certificate
* stores secret
* updates status

---

# Core Components

| Component           | Responsibility         |
| ------------------- | ---------------------- |
| CRD                 | Defines new API type   |
| API Server          | Registers API endpoint |
| etcd                | Stores CR objects      |
| CR                  | Desired state          |
| Controller/Operator | Watches and reconciles |
| Status Field        | Current state          |

---

# API Flow Internally

```text id="0md7a5"
kubectl apply -f cr.yaml
        ↓
API Server validates CR
        ↓
Stored in etcd
        ↓
Watch event triggered
        ↓
Controller receives event
        ↓
Controller reconciles
        ↓
Status updated
```

---

# Important Concept

## CRD Alone Does Nothing

CRD only:

* defines schema
* adds API endpoint

Actual automation comes from:

* Controller
* Operator

Without controller:

* CR becomes just a YAML object in etcd.

---

# Controller vs Operator

| Controller                     | Operator                            |
| ------------------------------ | ----------------------------------- |
| Basic automation logic         | Advanced domain-specific automation |
| Watches resources              | Watches + manages lifecycle         |
| Simpler                        | Smarter                             |
| Example: Deployment Controller | Example: Database Operator          |

---

# Real-World CRD Examples

| Tool                | Custom Resource |
| ------------------- | --------------- |
| Argo CD             | Application     |
| Istio               | VirtualService  |
| Prometheus Operator | ServiceMonitor  |
| Cert-Manager        | Certificate     |

---

# One-Line Interview Answer

> CRD extends the Kubernetes API, CR stores desired state, and controllers/operators continuously reconcile actual state with desired state using the reconciliation loop.
