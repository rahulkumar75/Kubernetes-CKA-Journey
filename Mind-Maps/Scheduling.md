# Kubernetes Scheduling — Complete Mind Map (0–3 Years Interview Revision)

```text
KUBERNETES SCHEDULING
│
├── Goal
│   │
│   ├── Decide Where Pods Run
│   ├── Optimize Resource Usage
│   ├── Ensure High Availability
│   └── Meet Application Requirements
│
├── 1. NodeSelector
│   │
│   ├── Purpose
│   │   └── Simple Node Selection
│   │
│   ├── Works Using Labels
│   │
│   ├── Example
│   │   ├── Node Label:
│   │   │   env=prod
│   │   │
│   │   └── Pod:
│   │       nodeSelector:
│   │         env: prod
│   │
│   ├── Characteristics
│   │   ├── Simple
│   │   ├── Exact Match Only
│   │   └── Hard Requirement
│   │
│   └── Limitation
│       └── No Complex Rules
│
├── 2. Node Affinity
│   │
│   ├── Purpose
│   │   └── Advanced Node Selection
│   │
│   ├── Types
│   │
│   ├── RequiredDuringScheduling
│   │   └── Hard Requirement
│   │
│   └── PreferredDuringScheduling
│       └── Soft Preference
│
│   Example:
│
│   affinity:
│     nodeAffinity:
│       requiredDuringScheduling...
│
│   Characteristics
│   │
│   ├── Supports AND / OR Logic
│   ├── Supports Expressions
│   ├── More Flexible
│   └── Production Preferred
│
├── 3. Taints & Tolerations
│   │
│   ├── Purpose
│   │   └── Keep Pods Away From Nodes
│   │
│   ├── Taint
│   │   └── Applied On Node
│   │
│   ├── Toleration
│   │   └── Applied On Pod
│   │
│   ├── Effects
│   │
│   ├── NoSchedule
│   │   └── New Pods Blocked
│   │
│   ├── PreferNoSchedule
│   │   └── Soft Block
│   │
│   └── NoExecute
│       └── Block + Evict
│
│   Example:
│
│   kubectl taint node node1 gpu=true:NoSchedule
│
│   tolerations:
│   - key: gpu
│     operator: Equal
│     value: "true"
│     effect: NoSchedule
│
│   Important
│   │
│   └── Toleration DOES NOT
│       force scheduling
│
├── 4. Pod Affinity
│   │
│   ├── Purpose
│   │   └── Place Pods Together
│   │
│   ├── Example
│   │   ├── Frontend Near Backend
│   │   └── Cache Near App
│   │
│   └── Improves
│       ├── Latency
│       └── Communication
│
├── 5. Pod Anti-Affinity
│   │
│   ├── Purpose
│   │   └── Separate Similar Pods
│   │
│   ├── Example
│   │   ├── Replica-1 → Node A
│   │   ├── Replica-2 → Node B
│   │   └── Replica-3 → Node C
│   │
│   └── Improves
│       ├── High Availability
│       └── Fault Tolerance
│
├── 6. Pod Priority
│   │
│   ├── Purpose
│   │   └── Importance Ranking
│   │
│   ├── PriorityClass
│   │
│   ├── High Value
│   │   └── Higher Importance
│   │
│   └── Low Value
│       └── Lower Importance
│
│   Example:
│
│   PriorityClass
│   value: 100000
│
├── 7. Preemption
│   │
│   ├── Cluster Full
│   │
│   ├── High Priority Pod Arrives
│   │
│   ├── Scheduler Looks For
│   │   Lower Priority Pods
│   │
│   ├── Evicts Them
│   │
│   └── Schedules
│       High Priority Pod
│
│   Flow
│   │
│   Cluster Full
│       ↓
│   Low Priority Pods Running
│       ↓
│   High Priority Pod Created
│       ↓
│   Scheduler Finds Victims
│       ↓
│   Victims Evicted
│       ↓
│   High Priority Pod Scheduled
│
├── Scheduler Decision Flow
│   │
│   ├── Pod Created
│   │
│   ├── Check Resources
│   │
│   ├── Check NodeSelector
│   │
│   ├── Check Node Affinity
│   │
│   ├── Check Taints
│   │
│   ├── Check Tolerations
│   │
│   ├── Check Pod Affinity
│   │
│   ├── Check Anti-Affinity
│   │
│   ├── Calculate Scores
│   │
│   └── Select Best Node
│
└── Interview Comparison
    │
    ├── NodeSelector
    │   └── Simple Attraction
    │
    ├── Node Affinity
    │   └── Advanced Attraction
    │
    ├── Taints
    │   └── Repulsion
    │
    ├── Tolerations
    │   └── Ignore Repulsion
    │
    ├── Pod Affinity
    │   └── Stay Together
    │
    ├── Pod Anti-Affinity
    │   └── Stay Apart
    │
    ├── Priority
    │   └── Importance
    │
    └── Preemption
        └── Evict Lower Priority Pods
```

# Ultimate Interview Memory Trick

```text
NODESELECTOR
↓
"Run on THIS node"

NODE AFFINITY
↓
"Prefer/Require THIS node"

TAINT
↓
"Stay away from THIS node"

TOLERATION
↓
"I can stay on that node"

POD AFFINITY
↓
"Stay together"

POD ANTI-AFFINITY
↓
"Stay apart"

PRIORITY
↓
"I am important"

PREEMPTION
↓
"Remove less important pods for me"
```

# 1-Minute Interview Answer

> Kubernetes scheduling starts by checking available resources on nodes. Then it evaluates NodeSelector and Node Affinity rules to find eligible nodes. Next it verifies Taints and Tolerations. After that it considers Pod Affinity and Anti-Affinity requirements. Finally, the scheduler scores the remaining nodes and selects the best one. If the cluster is full and a higher-priority pod arrives, Preemption can evict lower-priority pods to make room for the higher-priority workload.

This combined map covers almost all scheduling-related concepts commonly asked in **CKA**, **DevOps**, **Platform Engineer**, and **0–3 year Kubernetes interviews**.
