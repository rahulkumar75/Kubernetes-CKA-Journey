# Kubernetes Taints & Tolerations — Mind Map
```text
KUBERNETES TAINTS & TOLERATIONS
│
├── Purpose
│   │
│   ├── Control Pod Placement
│   ├── Protect Special Nodes
│   ├── Reserve Resources
│   └── Prevent Unwanted Scheduling
│
├── Concept
│   │
│   ├── Taint → Applied on Node
│   │   └── "Repels Pods"
│   │
│   └── Toleration → Applied on Pod
│       └── "Allows Pod to Ignore Taint"
│
├── Real World Example
│   │
│   ├── GPU Nodes
│   ├── Database Nodes
│   ├── High Memory Nodes
│   └── Dedicated Production Nodes
│
├── Taint Syntax
│   │
│   └── key=value:effect
│
│       Example:
│       gpu=true:NoSchedule
│
├── Create Taint
│   │
│   └── kubectl taint nodes node1 gpu=true:NoSchedule
│
├── Remove Taint
│   │
│   └── kubectl taint nodes node1 gpu=true:NoSchedule-
│
├── Effects
│   │
│   ├── NoSchedule
│   │   │
│   │   ├── New Pods NOT Scheduled
│   │   └── Existing Pods Stay Running
│   │
│   ├── PreferNoSchedule
│   │   │
│   │   ├── Soft Restriction
│   │   └── Scheduler Avoids Node If Possible
│   │
│   └── NoExecute
│       │
│       ├── New Pods NOT Scheduled
│       └── Existing Pods Evicted
│
├── Toleration Structure
│   │
│   ├── key
│   ├── operator
│   ├── value
│   └── effect
│
│   Example:
│
│   tolerations:
│   - key: "gpu"
│     operator: "Equal"
│     value: "true"
│     effect: "NoSchedule"
│
├── Operators
│   │
│   ├── Equal
│   │   └── Exact Match Required
│   │
│   └── Exists
│       └── Only Key Required
│
│       Example:
│
│       tolerations:
│       - key: "gpu"
│         operator: "Exists"
│
├── Scheduling Flow
│   │
│   ├── Pod Created
│   ├── Scheduler Checks Node Taints
│   ├── Compare Pod Tolerations
│   ├── Match Found?
│   │    │
│   │    ├── Yes → Schedule Pod
│   │    └── No → Pod Pending
│   │
│   └── Scheduler Selects Node
│
├── NoExecute Special Case
│   │
│   ├── Node Gets NoExecute Taint
│   ├── Existing Pods Checked
│   ├── Tolerating Pods Stay
│   └── Non-Tolerating Pods Evicted
│
├── tolerationSeconds
│   │
│   ├── Used Only With NoExecute
│   ├── Delay Eviction
│   └── Temporary Node Problems
│
│   Example:
│
│   tolerationSeconds: 300
│
├── Frequently Used Commands
│   │
│   ├── kubectl get nodes
│   ├── kubectl describe node node1
│   ├── kubectl taint nodes ...
│   ├── kubectl get pods -o wide
│   └── kubectl describe pod pod-name
│
├── Interview Differences
│   │
│   ├── NodeSelector
│   │   └── Attract Pod To Node
│   │
│   ├── Node Affinity
│   │   └── Advanced Attraction Rules
│   │
│   └── Taints
│       └── Repel Pods From Node
│
└── Common Interview Questions
    │
    ├── What is a taint?
    ├── What is a toleration?
    ├── Difference between NoSchedule and NoExecute?
    ├── Can toleration force scheduling?
    ├── What happens if toleration is missing?
    ├── What is tolerationSeconds?
    ├── Difference between taints and node affinity?
    └── How do you check node taints?
```

---

# 30-Second Interview Summary

```text
Taint = Applied on Node
Toleration = Applied on Pod

Taint repels pods.
Toleration allows pods to be scheduled on tainted nodes.

Effects:
NoSchedule      → Block new pods
PreferNoSchedule → Soft block
NoExecute       → Block + Evict existing pods

Toleration does NOT force scheduling.
It only allows scheduling if the scheduler chooses that node.
```

### One-Line Memory Trick

```text
Node says: "Stay Away!"      → Taint
Pod says: "I Can Tolerate"   → Toleration
```

This mind map is sufficient for **CKA revision and 0–3 year DevOps/Kubernetes interviews**, especially when combined with **NodeSelector**, **Node Affinity**, and **Pod Priority & Preemption**.
