## Kubernetes Pod Priority & Preemption — Mind Map

```text
Kubernetes Pod Priority & Preemption
│
├── Why Needed?
│   │
│   ├── Cluster resources are limited
│   ├── Critical applications must get resources first
│   └── Scheduler decides which pods are more important
│
├── PriorityClass
│   │
│   ├── Defines Pod Priority
│   ├── Cluster-wide Resource
│   ├── Integer Value
│   │
│   ├── Example
│   │   ├── low-priority     → 1000
│   │   ├── medium-priority  → 10000
│   │   └── high-priority    → 100000
│   │
│   └── Pod Usage
│       └── priorityClassName: high-priority
│
├── Scheduling Process
│   │
│   ├── Pod Created
│   ├── Scheduler Checks Resources
│   ├── Node Has Capacity?
│   │   │
│   │   ├── Yes → Schedule Pod
│   │   └── No  → Check Preemption
│   │
│   └── Higher Priority Pods Checked First
│
├── Preemption
│   │
│   ├── Triggered When
│   │   ├── Pod Cannot Be Scheduled
│   │   └── Pod Has Higher Priority
│   │
│   ├── Action
│   │   ├── Find Lower-Priority Pods
│   │   ├── Evict Them
│   │   └── Free Resources
│   │
│   └── Goal
│       └── Schedule Higher-Priority Pod
│
├── Important Condition
│   │
│   ├── Evicting Pods Must Free Enough Resources
│   │
│   ├── Example
│   │   ├── Available CPU = 500m
│   │   ├── High Priority Pod = 2000m
│   │   ├── Low Priority Pods = 3000m
│   │   └── Preemption Works ✅
│   │
│   └── Example
│       ├── Cluster Capacity = 8 CPU
│       ├── Pod Request = 500 CPU
│       └── Preemption Not Helpful ❌
│
├── Priority vs QoS
│   │
│   ├── Priority
│   │   └── Scheduling Importance
│   │
│   └── QoS
│       └── Eviction Importance During Resource Pressure
│
├── Common Errors
│   │
│   ├── cpu: "500"
│   │   └── Means 500 CPU cores ❌
│   │
│   ├── cpu: "500m"
│   │   └── Means 0.5 CPU core ✅
│   │
│   ├── Same Priority on All Pods
│   │   └── No Preemption Happens
│   │
│   └── Cluster Has Enough Resources
│       └── Scheduler Doesn't Need Preemption
│
├── Useful Commands
│   │
│   ├── kubectl get priorityclass
│   ├── kubectl describe priorityclass
│   ├── kubectl get pods
│   ├── kubectl describe pod <pod-name>
│   ├── kubectl top nodes
│   └── kubectl describe nodes
│
├── Troubleshooting
│   │
│   ├── Pending Pod?
│   │   └── kubectl describe pod
│   │
│   ├── Look For
│   │   ├── Insufficient CPU
│   │   ├── Insufficient Memory
│   │   ├── Untolerated Taints
│   │   └── Preemption Not Helpful
│   │
│   └── Check Node Allocatable Resources
│
└── Interview Questions
    │
    ├── What is PriorityClass?
    ├── What is Pod Preemption?
    ├── When does preemption occur?
    ├── Difference between Priority and QoS?
    ├── What does "Preemption is not helpful" mean?
    ├── Can equal-priority pods preempt each other?
    ├── Is PriorityClass namespaced?
    └── How do you assign priority to a Pod?
```

### 30-Second Interview Answer

```text
PriorityClass assigns importance to Pods using a numeric value.
When a high-priority Pod cannot be scheduled due to resource shortage,
the scheduler may preempt (evict) lower-priority Pods to free resources.
Preemption only happens if evicting lower-priority Pods can actually make
enough room for the higher-priority Pod.
```

### CKA Exam Focus

* Create and verify a `PriorityClass`
* Assign `priorityClassName` to Pods
* Understand scheduler events
* Troubleshoot `Pending` Pods
* Interpret:

  * `Insufficient cpu`
  * `Preemption is not helpful`
  * `Untolerated taints`
* Know the difference between **Priority** and **QoS Classes** (frequently asked together)
