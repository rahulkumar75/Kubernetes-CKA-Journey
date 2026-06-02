## 🎯 Kubernetes Interview Mind Map (0–3 Years)

```text
KUBERNETES
│
├── Cluster Basics
│   ├── cluster-info
│   ├── contexts
│   ├── nodes
│   └── api-resources
│
├── Workloads
│   ├── Pod
│   ├── ReplicaSet
│   ├── Deployment
│   ├── DaemonSet
│   ├── StatefulSet
│   └── Job/CronJob
│
├── Networking
│   ├── Service
│   ├── ClusterIP
│   ├── NodePort
│   ├── LoadBalancer
│   ├── Ingress
│   └── NetworkPolicy
│
├── Storage
│   ├── Volume
│   ├── PV
│   ├── PVC
│   ├── StorageClass
│   └── Dynamic Provisioning
│
├── Configuration
│   ├── ConfigMap
│   ├── Secret
│   └── Environment Variables
│
├── Scheduling
│   ├── NodeSelector
│   ├── Affinity
│   ├── Anti-Affinity
│   ├── Taints
│   └── Tolerations
│
├── Security
│   ├── ServiceAccount
│   ├── RBAC
│   ├── Role
│   ├── RoleBinding
│   └── Security Context
│
├── Scaling
│   ├── Manual Scaling
│   ├── HPA
│   ├── VPA
│   └── Cluster Autoscaler
│
├── Health Checks
│   ├── Liveness Probe
│   ├── Readiness Probe
│   └── Startup Probe
│
├── Troubleshooting
│   ├── logs
│   ├── describe
│   ├── exec
│   ├── events
│   ├── CrashLoopBackOff
│   ├── Pending Pods
│   └── ImagePullBackOff
│
└── Updates
    ├── Rolling Update
    ├── Rollback
    ├── Revision History
    └── Deployment Strategy
```

---

## 🔥 Most Asked Practical Questions (0–3 Years)

### Cluster

* How do you check the active cluster?
* Difference between cluster and context?
* How do you switch clusters?

### Pods & Deployments

* Difference between Pod, ReplicaSet, and Deployment?
* What happens during a rolling update?
* How do you rollback a deployment?

### Networking

* Difference between ClusterIP, NodePort, and LoadBalancer?
* How does a Service find Pods?
* What is Ingress?

### Storage

* Difference between PV and PVC?
* What happens if a PVC cannot bind?

### Scheduling

* Difference between NodeSelector and Node Affinity?
* Difference between Taints/Tolerations and Affinity?

### Troubleshooting

* Pod stuck in Pending. What do you check?
* Pod CrashLoopBackOff. What do you do?
* Service not reachable. What do you verify?

### Security

* What is RBAC?
* Difference between Role and ClusterRole?
* What is a ServiceAccount?

---

## ⭐ I'd prioritize:

1. Deployment & Rollback
2. Services & Ingress
3. ConfigMap & Secret
4. PV/PVC/StorageClass
5. HPA
6. RBAC
7. Troubleshooting (logs, events, describe)
8. Affinity + Taints/Tolerations
9. StatefulSet basics
10. Cluster operations (contexts, nodes, namespaces)

If you master these 10 areas with hands-on commands, you'll be well prepared for most Kubernetes interviews in the **0–3 year range**.
