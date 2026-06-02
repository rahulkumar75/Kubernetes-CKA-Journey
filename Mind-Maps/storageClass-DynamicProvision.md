# Kubernetes StorageClass & Dynamic Volume Provisioning — Mind Map (0–3 YOE Interview Revision)

```text
KUBERNETES STORAGECLASS & DYNAMIC VOLUME PROVISIONING
│
├── 1. Why StorageClass?
│   │
│   ├── Automates PV Creation
│   ├── Eliminates Manual PV Management
│   ├── Storage On-Demand
│   ├── Cloud Native Storage Management
│   │
│   └── Problem Solved
│       ├── Traditional Method
│       │   ├── Admin creates PV
│       │   └── User creates PVC
│       │
│       └── Dynamic Provisioning
│           ├── User creates PVC
│           └── Kubernetes creates PV automatically
│
├── 2. Core Components
│   │
│   ├── Pod
│   ├── PVC (Storage Request)
│   ├── StorageClass (Blueprint)
│   ├── Provisioner (Storage Driver)
│   └── PV (Actual Storage)
│
├── 3. Workflow
│   │
│   ├── Step 1 → Create StorageClass
│   ├── Step 2 → Create PVC
│   ├── Step 3 → PVC references StorageClass
│   ├── Step 4 → Provisioner Creates PV
│   ├── Step 5 → PV Binds PVC
│   └── Step 6 → Pod Uses PVC
│
├── 4. StorageClass Object
│   │
│   ├── API Version
│   │   └── storage.k8s.io/v1
│   │
│   ├── Important Fields
│   │
│   ├── provisioner
│   │   ├── AWS EBS CSI
│   │   ├── Azure Disk CSI
│   │   ├── GCE PD CSI
│   │   └── NFS CSI
│   │
│   ├── parameters
│   │   ├── disk type
│   │   ├── IOPS
│   │   ├── filesystem
│   │   └── performance settings
│   │
│   ├── reclaimPolicy
│   │   ├── Delete
│   │   └── Retain
│   │
│   ├── allowVolumeExpansion
│   │   ├── true
│   │   └── false
│   │
│   ├── volumeBindingMode
│   │   ├── Immediate
│   │   └── WaitForFirstConsumer
│   │
│   └── mountOptions
│
├── 5. Provisioner
│   │
│   ├── Responsible for PV Creation
│   │
│   ├── In-tree Drivers (Old)
│   │   ├── aws-ebs
│   │   ├── gce-pd
│   │   └── azure-disk
│   │
│   └── CSI Drivers (Current Standard)
│       ├── Container Storage Interface
│       ├── Vendor Independent
│       ├── Extensible
│       └── Recommended by Kubernetes
│
├── 6. Dynamic Provisioning
│   │
│   ├── PVC Created
│   ├── StorageClass Found
│   ├── CSI Driver Invoked
│   ├── Storage Allocated
│   ├── PV Generated
│   └── PVC Bound Automatically
│
├── 7. Reclaim Policies
│   │
│   ├── Delete
│   │   ├── PV Deleted
│   │   └── Backend Disk Deleted
│   │
│   ├── Retain
│   │   ├── Data Preserved
│   │   └── Manual Cleanup Needed
│   │
│   └── Recycle
│       └── Deprecated
│
├── 8. Volume Binding Modes
│   │
│   ├── Immediate
│   │   ├── PV Created Immediately
│   │   └── Default Behavior
│   │
│   └── WaitForFirstConsumer
│       ├── Wait Until Pod Scheduled
│       ├── Better AZ Placement
│       └── Common for Cloud Volumes
│
├── 9. Default StorageClass
│   │
│   ├── Cluster Wide Default
│   ├── Auto Used by PVC
│   ├── Annotation
│   │
│   └── Example
│       storageclass.kubernetes.io/is-default-class=true
│
├── 10. Volume Expansion
│   │
│   ├── Increase PVC Size
│   ├── Requires
│   │   └── allowVolumeExpansion=true
│   │
│   ├── Online Resize
│   └── Offline Resize
│
├── 11. Access Modes
│   │
│   ├── RWO
│   │   └── ReadWriteOnce
│   │
│   ├── ROX
│   │   └── ReadOnlyMany
│   │
│   ├── RWX
│   │   └── ReadWriteMany
│   │
│   └── RWOP
│       └── ReadWriteOncePod
│
├── 12. Common Commands
│   │
│   ├── kubectl get sc
│   ├── kubectl describe sc
│   ├── kubectl get pvc
│   ├── kubectl get pv
│   ├── kubectl describe pvc
│   ├── kubectl describe pv
│   └── kubectl edit pvc
│
├── 13. Production Use Cases
│   │
│   ├── Database Storage
│   │   ├── MySQL
│   │   ├── PostgreSQL
│   │   └── MongoDB
│   │
│   ├── StatefulSets
│   ├── Logging Storage
│   ├── Shared NFS Storage
│   ├── EBS Volumes
│   ├── Azure Disks
│   └── Persistent Application Data
│
├── 14. Troubleshooting
│   │
│   ├── PVC Pending
│   │   ├── StorageClass Missing
│   │   ├── CSI Driver Missing
│   │   ├── Insufficient Capacity
│   │   └── Access Mode Mismatch
│   │
│   ├── PV Not Bound
│   ├── Volume Attach Errors
│   ├── CSI Driver Logs
│   └── Event Analysis
│
└── 15. Interview Questions
    │
    ├── What is a StorageClass?
    ├── Why use Dynamic Provisioning?
    ├── Difference between PV, PVC and StorageClass?
    ├── What is a Provisioner?
    ├── What is CSI?
    ├── Immediate vs WaitForFirstConsumer?
    ├── Delete vs Retain reclaim policy?
    ├── How does PVC trigger PV creation?
    ├── How to make a StorageClass default?
    ├── How does volume expansion work?
    ├── Why is PVC stuck in Pending?
    └── How is storage managed in StatefulSets?
```

---

# Interview-Ready 30-Second Answer

**"StorageClass is a Kubernetes resource that enables dynamic provisioning of Persistent Volumes. Instead of manually creating PVs, an administrator defines a StorageClass with a provisioner such as an AWS EBS CSI driver. When a user creates a PVC referencing that StorageClass, Kubernetes automatically provisions a matching PV and binds it to the claim. StorageClass also defines properties like reclaim policy, volume expansion, and volume binding mode, making storage management scalable and cloud-native."**

### Quick Memory Formula

```text
StorageClass
     ↓
PVC Created
     ↓
CSI Provisioner
     ↓
PV Created
     ↓
PVC Bound
     ↓
Pod Uses Storage
```

For CKA and 0–3 YOE interviews, remember:

**PV = Storage**
→ **PVC = Request**
→ **StorageClass = Blueprint**
→ **CSI Driver = Creator**
→ **Dynamic Provisioning = Automatic PV Creation**.
