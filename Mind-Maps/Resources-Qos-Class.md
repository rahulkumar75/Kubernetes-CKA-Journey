# Kubernetes QoS Classes (Guaranteed, Burstable, BestEffort) — Mind Map

```text
Kubernetes QoS (Quality of Service)
│
├── Purpose
│   │
│   ├── Pod Priority During Resource Pressure
│   ├── Eviction Decision
│   ├── Memory Pressure Handling
│   └── Node Resource Management
│
├── QoS Classes
│   │
│   ├── 1. Guaranteed ⭐ Highest Priority
│   │   │
│   │   ├── Condition
│   │   │   ├── Requests defined
│   │   │   ├── Limits defined
│   │   │   └── Request = Limit
│   │   │
│   │   ├── Example
│   │   │   ├── CPU Request = 500m
│   │   │   ├── CPU Limit = 500m
│   │   │   ├── Memory Request = 512Mi
│   │   │   └── Memory Limit = 512Mi
│   │   │
│   │   ├── Eviction Order
│   │   │   └── Evicted Last
│   │   │
│   │   └── Production Use
│   │       ├── Databases
│   │       ├── Critical APIs
│   │       └── Stateful Applications
│   │
│   ├── 2. Burstable ⭐ Most Common
│   │   │
│   │   ├── Condition
│   │   │   ├── Requests defined
│   │   │   └── Request ≠ Limit
│   │   │
│   │   ├── Example
│   │   │   ├── CPU Request = 100m
│   │   │   ├── CPU Limit = 500m
│   │   │   ├── Memory Request = 256Mi
│   │   │   └── Memory Limit = 1Gi
│   │   │
│   │   ├── Eviction Order
│   │   │   └── Evicted After BestEffort
│   │   │
│   │   └── Production Use
│   │       ├── Web Applications
│   │       ├── Backend Services
│   │       └── Microservices
│   │
│   └── 3. BestEffort ⭐ Lowest Priority
│       │
│       ├── Condition
│       │   └── No Requests & No Limits
│       │
│       ├── Example
│       │   └── Empty Resources Block
│       │
│       ├── Eviction Order
│       │   └── Evicted First
│       │
│       └── Production Use
│           ├── Testing
│           ├── Temporary Jobs
│           └── Non-Critical Workloads
│
├── Eviction Priority
│   │
│   ├── Node Memory Pressure
│   │
│   ├── BestEffort
│   │      ↓
│   ├── Burstable
│   │      ↓
│   └── Guaranteed
│          ↓
│      Evicted Last
│
├── How Kubernetes Assigns QoS
│   │
│   ├── Check Requests?
│   ├── Check Limits?
│   ├── Compare Request & Limit?
│   │
│   ├── Equal → Guaranteed
│   ├── Different → Burstable
│   └── Missing → BestEffort
│
├── Verification
│   │
│   ├── kubectl get pod
│   ├── kubectl describe pod
│   │
│   └── Look For
│       └── QoS Class:
│           ├── Guaranteed
│           ├── Burstable
│           └── BestEffort
│
├── Production Design
│   │
│   ├── Database → Guaranteed
│   ├── API Servers → Guaranteed/Burstable
│   ├── Frontend → Burstable
│   ├── Workers → Burstable
│   └── Test Pods → BestEffort
│
└── Interview Questions
    │
    ├── What are Kubernetes QoS Classes?
    ├── How many QoS Classes exist?
    ├── Difference between Guaranteed and Burstable?
    ├── Which Pod is evicted first?
    ├── Which Pod is safest during memory pressure?
    ├── How is QoS calculated?
    └── How to check QoS class?
```

---

# Quick YAML Examples

## 1️⃣ Guaranteed

```yaml
resources:
  requests:
    cpu: "500m"
    memory: "512Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

✅ Request = Limit
✅ Highest Priority

---

## 2️⃣ Burstable

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "256Mi"
  limits:
    cpu: "500m"
    memory: "1Gi"
```

✅ Most common in production

---

## 3️⃣ BestEffort

```yaml
containers:
- name: nginx
  image: nginx
```

✅ No requests
✅ No limits

---

# QoS Comparison Table

| Feature          | Guaranteed    | Burstable    | BestEffort |
| ---------------- | ------------- | ------------ | ---------- |
| Requests         | Yes           | Yes          | No         |
| Limits           | Yes           | Optional/Yes | No         |
| Request = Limit  | Yes           | No           | No         |
| Priority         | Highest       | Medium       | Lowest     |
| Evicted First?   | ❌             | ⚠️ Sometimes | ✅ Yes      |
| Production Usage | Critical Apps | Most Apps    | Testing    |

---

# Visual Memory Trick

```text
Memory Pressure on Node

Guaranteed 🏆
     ↑
Burstable 🚀
     ↑
BestEffort ⚠️

Eviction Starts Here
```

---

# CKA / Interview Golden Answer

> Kubernetes provides three QoS classes: Guaranteed, Burstable, and BestEffort. QoS determines which Pods are evicted first during node resource pressure. BestEffort Pods are evicted first, Burstable next, and Guaranteed Pods last. Guaranteed Pods have requests equal to limits, while Burstable Pods have requests and limits that differ. BestEffort Pods have neither requests nor limits.

## Commands to Remember

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
QoS Class: Guaranteed
```

and

```bash
kubectl get pod -o yaml
```

to inspect requests and limits.

🎯 For 0–3 years DevOps/Kubernetes interviews, **Requests + Limits + QoS + OOMKilled + CPU Throttling** form one complete resource-management topic and are asked very frequently.
