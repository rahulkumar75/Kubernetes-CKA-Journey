# Kubernetes Network Policy Mind Map (0–3 Years DevOps/CKA Interview Revision)

```text
Kubernetes Network Policy
│
├── Purpose
│   ├── Pod Firewall
│   ├── Control Pod-to-Pod Traffic
│   ├── Zero Trust Networking
│   ├── Micro-Segmentation
│   └── Reduce Attack Surface
│
├── Prerequisites
│   ├── CNI Must Support NetworkPolicy
│   ├── Calico
│   ├── Cilium
│   ├── Antrea
│   └── Without Supported CNI → No Effect
│
├── Core Components
│   │
│   ├── podSelector
│   │   ├── Select Destination Pods
│   │   └── Policy Applies To These Pods
│   │
│   ├── policyTypes
│   │   ├── Ingress
│   │   ├── Egress
│   │   └── Both
│   │
│   ├── ingress
│   │   ├── Incoming Traffic Rules
│   │   ├── from
│   │   └── ports
│   │
│   └── egress
│       ├── Outgoing Traffic Rules
│       ├── to
│       └── ports
│
├── Selectors
│   │
│   ├── podSelector
│   │   └── Select Pods
│   │
│   ├── namespaceSelector
│   │   └── Select Namespaces
│   │
│   ├── ipBlock
│   │   └── CIDR Based Rules
│   │
│   └── matchExpressions
│       ├── In
│       ├── NotIn
│       ├── Exists
│       └── DoesNotExist
│
├── Default Behavior
│   ├── No Policy Applied
│   │   └── Allow All Traffic
│   │
│   ├── Policy Applied
│   │   └── Allow Only Defined Rules
│   │
│   └── NetworkPolicy is Allow-List Based
│
├── Common Policies
│   │
│   ├── Default Deny All
│   │   └── podSelector: {}
│   │
│   ├── Frontend → Backend
│   │   └── Allow TCP 8080
│   │
│   ├── Backend → Database
│   │   └── Allow TCP 3306
│   │
│   ├── Allow DNS
│   │   ├── UDP 53
│   │   └── TCP 53
│   │
│   └── Monitoring Access
│       └── Prometheus → Application
│
├── Enterprise 3-Tier Architecture
│   │
│   ├── Frontend
│   │   └── Web/UI Pods
│   │
│   ├── Backend
│   │   └── API Pods
│   │
│   ├── Database
│   │   └── MySQL/PostgreSQL
│   │
│   └── Flow
│       Frontend
│           ↓
│       Backend
│           ↓
│       Database
│
├── Namespace Isolation
│   ├── Dev Namespace
│   ├── QA Namespace
│   ├── Prod Namespace
│   └── Prevent Cross-Namespace Access
│
├── Interview Concepts
│   │
│   ├── podSelector = Destination Pods
│   ├── from/to = Source/Destination Rules
│   ├── ports = Allowed Ports
│   ├── Ingress = Incoming Traffic
│   ├── Egress = Outgoing Traffic
│   └── Policies are Additive
│
├── Troubleshooting
│   │
│   ├── kubectl get netpol
│   ├── kubectl describe netpol
│   ├── Verify Pod Labels
│   ├── Verify Namespace Labels
│   ├── Check CNI Support
│   └── Test Using BusyBox Pod
│
├── CKA Exam Tips
│   ├── Create Default Deny First
│   ├── Allow Required Traffic Only
│   ├── Verify Labels Carefully
│   ├── Test Connectivity
│   └── Remember DNS Rules
│
└── Interview One-Liner
    └── "NetworkPolicy acts as a firewall for Kubernetes Pods,
        controlling ingress and egress traffic using labels,
        namespaces, and ports."
```

## 0–3 Year Interview Must-Remember Points

### 1. What is NetworkPolicy?

> NetworkPolicy is a Kubernetes resource used to control pod-to-pod communication, similar to a firewall for pods.

### 2. Does it work by default?

> No. A CNI such as Calico or Cilium must support NetworkPolicy.

### 3. Difference between Ingress and Egress?

| Type    | Controls                  |
| ------- | ------------------------- |
| Ingress | Incoming traffic to pod   |
| Egress  | Outgoing traffic from pod |

### 4. Key Rule to Remember

```text
podSelector = Destination Pod
from/to      = Source/Destination
ports        = Allowed Ports
```

### 5. Real-World Example

```text
Frontend → Backend → MySQL

Allow:
Frontend → Backend:8080
Backend → MySQL:3306

Block:
Frontend → MySQL
Random Pod → MySQL
```

This level of understanding is usually enough to confidently answer most Kubernetes NetworkPolicy questions in **CKA, Junior DevOps, and 0–3 year DevOps interviews**.
