# 🌳 Kubernetes Storage (PV/PVC/StorageClass) — Mind Map

```text
KUBERNETES STORAGE
│
├── Why Storage?
│   │
│   ├── Containers are Ephemeral
│   ├── Pod restart → data loss
│   └── Need Persistent Storage
│
├── Volume Types
│   │
│   ├── emptyDir
│   │   ├── Created with Pod
│   │   ├── Deleted with Pod
│   │   └── Cache / Temp Files
│   │
│   ├── hostPath
│   │   ├── Uses Node Filesystem
│   │   ├── Node-specific
│   │   └── Lab / Testing
│   │
│   └── Persistent Storage
│
├── PersistentVolume (PV)
│   │
│   ├── Actual Storage Resource
│   ├── Created by Admin
│   ├── Cluster-wide Resource
│   │
│   ├── Components
│   │   ├── capacity
│   │   ├── accessModes
│   │   ├── storageClassName
│   │   └── reclaimPolicy
│   │
│   └── Examples
│       ├── AWS EBS
│       ├── EFS
│       ├── NFS
│       ├── Ceph
│       └── hostPath
│
├── PersistentVolumeClaim (PVC)
│   │
│   ├── Storage Request
│   ├── Created by Developer
│   │
│   ├── Defines
│   │   ├── Size
│   │   ├── Access Mode
│   │   └── StorageClass
│   │
│   └── Binds to Matching PV
│
├── StorageClass (SC)
│   │
│   ├── Dynamic Provisioning
│   ├── Auto-Creates PV
│   │
│   ├── Production Examples
│   │   ├── gp3 (AWS)
│   │   ├── efs-sc
│   │   ├── managed-csi
│   │   └── local-path
│   │
│   └── Uses CSI Driver
│
├── Binding Flow
│   │
│   ├── Static Provisioning
│   │
│   │   Admin
│   │     ↓
│   │    PV
│   │     ↓
│   │    PVC
│   │     ↓
│   │    Pod
│   │
│   └── Dynamic Provisioning
│
│       PVC
│        ↓
│    StorageClass
│        ↓
│   CSI Driver
│        ↓
│       PV
│        ↓
│       Pod
│
├── Access Modes
│   │
│   ├── RWO
│   │   └── ReadWriteOnce
│   │       One Node Read/Write
│   │
│   ├── ROX
│   │   └── ReadOnlyMany
│   │
│   └── RWX
│       └── ReadWriteMany
│           Multiple Nodes RW
│
├── Reclaim Policy
│   │
│   ├── Retain
│   │   └── Keep Data
│   │
│   ├── Delete
│   │   └── Delete Storage
│   │
│   └── Recycle
│       └── Deprecated
│
├── Production Storage
│   │
│   ├── Database
│   │   ├── PostgreSQL
│   │   ├── MySQL
│   │   └── MongoDB
│   │
│   ├── Jenkins
│   ├── Elasticsearch
│   ├── Redis Persistence
│   └── Shared Uploads
│
├── Storage Backends
│   │
│   ├── AWS
│   │   ├── EBS (RWO)
│   │   └── EFS (RWX)
│   │
│   ├── Azure
│   │   ├── Azure Disk
│   │   └── Azure Files
│   │
│   ├── GCP
│   │   └── Persistent Disk
│   │
│   └── On-Prem
│       ├── NFS
│       ├── Ceph
│       └── Longhorn
│
├── Stateful Workloads
│   │
│   ├── StatefulSet
│   ├── volumeClaimTemplates
│   └── One PVC per Replica
│
├── Troubleshooting
│   │
│   ├── PVC Pending
│   │   ├── No PV
│   │   ├── SC Mismatch
│   │   ├── Size Mismatch
│   │   └── AccessMode Mismatch
│   │
│   ├── Pod Pending
│   │   └── PVC Not Bound
│   │
│   └── Commands
│       ├── kubectl get pv
│       ├── kubectl get pvc
│       ├── kubectl get sc
│       ├── kubectl describe pvc
│       └── kubectl describe pv
│
└── CKA Memory Trick

    PV  = Actual Storage
    PVC = Storage Request
    SC  = Auto Storage Factory

    Pod
     ↓
    PVC
     ↓
     PV
```

## ⚡ 30-Second Exam Revision

```text
Storage Flow

Pod
 ↓
PVC (request)
 ↓
StorageClass (optional)
 ↓
PV (actual storage)
 ↓
Disk (EBS/EFS/NFS)

Static:
PV → PVC → Pod

Dynamic:
PVC → SC → PV → Pod

RWO = One Node
RWX = Many Nodes

hostPath = Node Local
EBS = Persistent Block Storage
EFS = Shared Storage

PVC Pending?
→ describe pvc

Pod Pending?
→ check PVC status
```

---

## For 0–3 years DevOps/Kubernetes interviews.

> Your current PV/PVC/StorageClass knowledge covers about 70–80% of what is usually asked.
---

# 🎯 Level 1 (Must Know)

### 1. Difference Between PV, PVC, StorageClass

Interview:

> Explain PV, PVC and StorageClass.

Expected answer:

```text
PV = Actual storage resource
PVC = Request for storage
StorageClass = Dynamic provisioning template

PVC requests storage.
StorageClass provisions PV automatically.
Pod mounts PVC.
```

---

### 2. Static vs Dynamic Provisioning

```text
Static:
Admin creates PV manually.

Dynamic:
StorageClass creates PV automatically.
```

Real-world:

```text
99% of production clusters use dynamic provisioning.
```

---

### 3. Access Modes

| Mode | Meaning              |
| ---- | -------------------- |
| RWO  | One node RW          |
| ROX  | Many nodes Read Only |
| RWX  | Many nodes RW        |

Interview favorite:

> Why can't EBS be mounted by multiple nodes?

Answer:

```text
EBS supports RWO only.
For shared storage use EFS.
```

---

### 4. Reclaim Policies

```text
Retain
Delete
Recycle (deprecated)
```

Common question:

> What happens when PVC is deleted?

Answer:

Depends on reclaim policy.

---

# 🎯 Level 2 (Frequently Asked in Real Interviews)

### 5. CSI Driver

Most freshers skip this.

Interview:

> How does Kubernetes provision EBS automatically?

Answer:

```text
StorageClass
↓
CSI Driver
↓
Cloud API
↓
Creates Disk
↓
Creates PV
```

Know the term:

**Container Storage Interface (CSI)**

---

### 6. EBS vs EFS

Very common in AWS interviews.

| EBS           | EFS                 |
| ------------- | ------------------- |
| Block Storage | Shared File Storage |
| RWO           | RWX                 |
| Single Node   | Multi Node          |
| Database      | Shared Files        |

Use cases:

```text
PostgreSQL → EBS
Jenkins Shared Home → EFS
Uploads → EFS
```

---

### 7. StatefulSet + PVC

Interview:

> Why StatefulSet for databases?

Answer:

```text
Stable identity
Dedicated storage
One PVC per replica
```

Know:

```yaml
volumeClaimTemplates
```

---

### 8. Volume Expansion

Production question:

> Disk becomes full. What will you do?

Example:

```yaml
resources:
  requests:
    storage: 20Gi
```

Change:

```yaml
resources:
  requests:
    storage: 50Gi
```

Then:

```bash
kubectl edit pvc
```

Know that:

* StorageClass must support expansion
* CSI driver must support expansion

---

# 🎯 Level 3 (Makes You Stand Out)

### 9. Troubleshooting Flow

Interview:

> PVC is Pending. How do you troubleshoot?

Answer:

```bash
kubectl get pvc
kubectl describe pvc
kubectl get pv
kubectl get sc
kubectl get events
```

Check:

* StorageClass exists?
* Access mode match?
* Capacity match?
* CSI driver running?

---

### 10. Multi-Attach Error

AWS interview favorite.

Example:

```text
Volume already attached to node-1
Pod scheduled on node-2
```

Error:

```text
Multi-Attach Error
```

Reason:

```text
EBS = RWO
```

---

### 11. Backup Strategy

Interview:

> Is PVC a backup?

Answer:

```text
No.

PVC provides persistence.

Backup requires:
- EBS Snapshots
- Velero
- Database Dumps
```

Very good answer.

---

### 12. Real Production Architecture

Be able to explain:

```text
PostgreSQL
 ↓
PVC (20Gi)
 ↓
StorageClass gp3
 ↓
EBS Volume
 ↓
CSI Driver
```

This sounds like real experience.

---

# ⭐ Questions I Would Ask a Junior DevOps Engineer

1. Difference between PV and PVC?
2. Why StorageClass?
3. What is CSI?
4. Difference between EBS and EFS?
5. Why StatefulSet for databases?
6. PVC Pending troubleshooting?
7. What is ReadWriteOnce?
8. Can multiple pods use same PVC?
9. What happens if node dies?
10. Is PVC a backup?

If you answer these confidently, you're already above many 0–3 year candidates.

---

# 📌 My Recommendation for Your CKA + DevOps Roadmap

After PV/PVC/StorageClass, study in this order:

```text
Storage (Done)
↓
StatefulSet + volumeClaimTemplates
↓
Headless Service
↓
CSI Drivers
↓
EBS vs EFS
↓
Velero Backup
↓
Production PostgreSQL on Kubernetes
```

These topics connect directly to your current role as a Junior DevOps Engineer and are frequently discussed in Kubernetes-focused interviews.
