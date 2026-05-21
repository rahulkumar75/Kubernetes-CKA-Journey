# API Extensions in Kubernetes

## What are API Extensions?

API Extensions allow Kubernetes to:

* add new resource types
* extend Kubernetes API dynamically
* support custom platforms/tools

Without modifying Kubernetes core source code.

---

# Why API Extensions Exist?

Kubernetes has built-in resources like:

* Pod
* Deployment
* Service

But real-world platforms need custom resources like:

* Database
* KafkaCluster
* Application
* Certificate

Kubernetes API Extensions make this possible.

---

# Main API Extension Methods

| Method                         | Purpose               |
| ------------------------------ | --------------------- |
| CRD (CustomResourceDefinition) | Add custom resources  |
| Aggregated API Server          | Add fully custom APIs |

---

# 1. CRD-Based Extension (Most Common)

This is the most widely used method.

You create:

```yaml id="jfdj46"
kind: CustomResourceDefinition
```

Kubernetes automatically:

* creates new API endpoints
* stores objects in etcd
* exposes via kubectl

---

# Flow

```text id="4oq7wx"
CRD Created
      ↓
API Server registers new endpoint
      ↓
Custom Resource API available
      ↓
Users create CR objects
      ↓
Controller/Operator handles logic
```

---

# Example

## CRD

```yaml id="j6wb2d"
group: mycompany.com
versions:
  - name: v1
names:
  kind: Database
  plural: databases
```

---

## New API Endpoint

```text id="xbxg0d"
/apis/mycompany.com/v1/databases
```

---

## Use with kubectl

```bash id="qu9d61"
kubectl get databases
```

---

# Internally What Happens?

Kubernetes API Server contains:

```text id="fjlwmk"
API Extension Server
```

which dynamically handles:

* CRDs
* custom APIs

---

# Kubernetes API Architecture

```text id="ut8on9"
                kube-apiserver
                       |
        --------------------------------
        |                              |
        v                              v
 Built-in APIs                 API Extensions
 (Pods, Deployments)           (CRDs, Aggregated APIs)
```

---

# 2. Aggregated API Server

Advanced extension mechanism.

Instead of simple CRDs:

* you build your own API server.

Useful when:

* custom authentication needed
* custom storage needed
* non-etcd backend needed
* advanced API behavior required

---

# Architecture

```text id="4d0r7u"
kubectl
   ↓
kube-apiserver
   ↓
Aggregated API Server
   ↓
Custom backend/service
```

---

# Difference: CRD vs Aggregated API

| Feature           | CRD     | Aggregated API |
| ----------------- | ------- | -------------- |
| Easy setup        | Yes     | Complex        |
| Uses etcd         | Yes     | Optional       |
| Custom logic      | Limited | Full control   |
| Most common       | Yes     | Rare           |
| Used by operators | Yes     | Sometimes      |

---

# API Groups

Kubernetes organizes APIs into groups.

---

# Built-in Examples

| Resource   | API Group |
| ---------- | --------- |
| Pod        | core/v1   |
| Deployment | apps/v1   |
| Job        | batch/v1  |

---

# Custom Example

```text id="wq0cns"
mycompany.com/v1
```

---

# Check Available APIs

## List API groups

```bash id="r9q74g"
kubectl api-versions
```

---

## List all resources

```bash id="dz9q9i"
kubectl api-resources
```

---

# Real-World Examples

Many Kubernetes tools extend APIs using CRDs.

---

# Examples

| Tool                | Custom Resource |
| ------------------- | --------------- |
| Argo CD             | Application     |
| Cert-Manager        | Certificate     |
| Istio               | VirtualService  |
| Prometheus Operator | ServiceMonitor  |

---

# API Discovery

After CRD creation:

```bash id="jlwm5k"
kubectl api-resources
```

shows:

```text id="rsg63y"
NAME         SHORTNAMES   APIVERSION
databases    db           mycompany.com/v1
```

---

# Important Internal Components

| Component      | Responsibility           |
| -------------- | ------------------------ |
| kube-apiserver | Handles APIs             |
| API Extensions | Dynamic API registration |
| etcd           | Stores custom resources  |
| Controller     | Business logic           |
| OpenAPI Schema | Validation               |

---

# Important Concept

## CRD Only Extends API

CRD does NOT:

* create infrastructure
* automate workflows

Automation comes from:

* controllers/operators

---

# Example Full Flow

```text id="m4o8ys"
Create CRD
      ↓
API endpoint created
      ↓
User creates CR
      ↓
Stored in etcd
      ↓
Controller watches CR
      ↓
Controller reconciles state
```

---

# Interview-Level Understanding

## Why API Extensions are Powerful?

Because Kubernetes becomes:

* platform extensible
* cloud-native framework
* automation platform

instead of just:

* container orchestrator

---

# One-Line Interview Answer

> Kubernetes API Extensions allow adding custom APIs and resources dynamically using CRDs or Aggregated API Servers, enabling platforms and operators to extend Kubernetes functionality without modifying Kubernetes core code.
