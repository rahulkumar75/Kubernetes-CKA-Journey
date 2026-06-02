# Kubernetes Resources (Requests, Limits, CPU Throttling, OOMKilled) — Mind Map

```text
Kubernetes Resource Management
│
├── 1. Requests
│   │
│   ├── Purpose
│   │   └── Minimum resources guaranteed
│   │
│   ├── Used By
│   │   └── Scheduler
│   │
│   ├── Examples
│   │   ├── cpu: 100m
│   │   └── memory: 256Mi
│   │
│   └── Interview
│       └── Node selection is based on requests
│
├── 2. Limits
│   │
│   ├── Purpose
│   │   └── Maximum resource allowed
│   │
│   ├── CPU Limit
│   │   └── Throttled if exceeded
│   │
│   ├── Memory Limit
│   │   └── OOMKilled if exceeded
│   │
│   └── Interview
│       └── Limits protect node resources
│
├── 3. CPU Resources
│   │
│   ├── Unit
│   │   └── Millicores (m)
│   │
│   ├── Examples
│   │   ├── 1000m = 1 CPU
│   │   ├── 500m = 0.5 CPU
│   │   └── 100m = 0.1 CPU
│   │
│   ├── Behavior
│   │   └── CPU limit exceeded
│   │       └── CPU Throttling
│   │
│   └── Result
│       ├── Slow application
│       ├── High latency
│       └── No container restart
│
├── 4. Memory Resources
│   │
│   ├── Units
│   │   ├── Mi
│   │   └── Gi
│   │
│   ├── Behavior
│   │   └── Memory limit exceeded
│   │
│   └── Result
│       ├── OOMKilled
│       ├── Container Restart
│       └── CrashLoopBackOff
│
├── 5. CPU Throttling
│   │
│   ├── Cause
│   │   └── CPU usage > CPU limit
│   │
│   ├── Symptoms
│   │   ├── Slow response
│   │   ├── Increased latency
│   │   └── No pod crash
│   │
│   ├── Detection
│   │   ├── kubectl top pods
│   │   ├── kubectl top nodes
│   │   └── Metrics Server
│   │
│   └── Interview
│       └── CPU limit causes throttling, not killing
│
├── 6. OOMKilled
│   │
│   ├── Cause
│   │   └── Memory usage > Memory limit
│   │
│   ├── Detection
│   │   ├── kubectl describe pod
│   │   └── Last State: OOMKilled
│   │
│   └── Interview
│       └── Linux OOM Killer terminates container
│
├── 7. Metrics Server
│   │
│   ├── Purpose
│   │   └── Collect CPU/Memory metrics
│   │
│   ├── Commands
│   │   ├── kubectl top nodes
│   │   └── kubectl top pods
│   │
│   ├── Common Issue
│   │   └── Metrics API not available
│   │
│   └── Kind Fix
│       ├── --kubelet-insecure-tls
│       └── --kubelet-preferred-address-types=InternalIP
│
├── 8. Troubleshooting Flow
│   │
│   ├── Pod Restarting?
│   │   └── kubectl describe pod
│   │
│   ├── OOMKilled?
│   │   └── Increase memory limit
│   │
│   ├── Slow Pod?
│   │   └── Check CPU throttling
│   │
│   └── No Metrics?
│       └── Verify Metrics Server
│
└── 9. Interview Questions (0-3 Years)
    │
    ├── Difference between requests and limits?
    │
    ├── What happens if CPU limit is exceeded?
    │   └── Throttling
    │
    ├── What happens if Memory limit is exceeded?
    │   └── OOMKilled
    │
    ├── Why is CPU not killing containers?
    │
    ├── How do you monitor CPU/Memory?
    │   └── kubectl top
    │
    ├── What is Metrics Server?
    │
    └── How do you troubleshoot resource issues?
```

## 0–3 Year Interview Must Remember (One-Liners)

| Topic                 | Key Point                |
| --------------------- | ------------------------ |
| Request               | Used by Scheduler        |
| Limit                 | Maximum Allowed Resource |
| CPU Limit Exceeded    | Throttled                |
| Memory Limit Exceeded | OOMKilled                |
| CPU                   | Never kills container    |
| Memory                | Can kill container       |
| OOMKilled Check       | `kubectl describe pod`   |
| Resource Usage        | `kubectl top pods`       |
| Node Usage            | `kubectl top nodes`      |
| Metrics Collection    | Metrics Server           |

### Golden Interview Answer

> **Requests decide where a Pod can be scheduled. Limits decide how much resource a Pod can consume. If CPU exceeds the limit, Kubernetes throttles it. If memory exceeds the limit, the Linux OOM Killer terminates the container, resulting in OOMKilled.** 🚀

For a **0–3 year DevOps/Kubernetes interview**, this topic is complete when combined with the next topic: **QoS Classes (Guaranteed, Burstable, BestEffort)**, which is frequently asked together with Requests and Limits.
