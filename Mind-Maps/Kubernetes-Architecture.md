# ☸️ Kubernetes Architecture Mind Map (0–3 Years Interview Revision)

```text
                                   ☸️ KUBERNETES ARCHITECTURE
                                                │
        ┌───────────────────────────────────────┼───────────────────────────────────────┐
        │                                       │                                       │
        ▼                                       ▼                                       ▼
 ┌──────────────┐                      ┌─────────────────┐                    ┌────────────────┐
 │ Control Plane │                      │ Worker Node     │                    │ External Users │
 └──────────────┘                      └─────────────────┘                    └────────────────┘
        │                                       │                                       │
        │                                       │                                       │
        ▼                                       ▼                                       ▼

══════════════════════════════════════════════════════════════════════════════════════════════

🧠 CONTROL PLANE (Brain of Cluster)

├── API Server (kube-apiserver)
│   ├─ Entry Point of Cluster
│   ├─ Receives kubectl requests
│   ├─ Authentication
│   ├─ Authorization (RBAC)
│   ├─ Admission Controllers
│   └─ Communicates with etcd

├── ETCD
│   ├─ Key-Value Database
│   ├─ Stores Cluster State
│   ├─ Stores Secrets
│   ├─ Stores ConfigMaps
│   └─ Single Source of Truth

├── Scheduler
│   ├─ Watches Pending Pods
│   ├─ Selects Best Node
│   ├─ Resource Calculation
│   ├─ Affinity / Anti-Affinity
│   └─ Taints & Tolerations

├── Controller Manager
│   ├─ Deployment Controller
│   ├─ ReplicaSet Controller
│   ├─ Node Controller
│   ├─ Endpoint Controller
│   └─ Namespace Controller

└── Cloud Controller Manager
    ├─ Load Balancer
    ├─ Node Management
    ├─ Route Management
    └─ Cloud Integration (AWS/Azure/GCP)

══════════════════════════════════════════════════════════════════════════════════════════════

⚙️ WORKER NODE

├── Kubelet
│   ├─ Node Agent
│   ├─ Talks to API Server
│   ├─ Creates Pods
│   ├─ Monitors Containers
│   └─ Reports Node Status

├── Container Runtime
│   ├─ containerd
│   ├─ CRI-O
│   └─ Runs Containers

├── Kube-Proxy
│   ├─ Service Networking
│   ├─ Maintains iptables/IPVS
│   ├─ Load Balancing
│   └─ Pod Communication

└── Pods
    ├─ Single Container Pod
    ├─ Multi Container Pod
    ├─ Init Containers
    └─ Sidecar Containers

══════════════════════════════════════════════════════════════════════════════════════════════

🌐 NETWORKING

├── Pod Network
│   ├─ Every Pod gets IP
│   └─ Pod-to-Pod Communication

├── Service
│   ├─ ClusterIP
│   ├─ NodePort
│   ├─ LoadBalancer
│   └─ ExternalName

├── Ingress
│   ├─ HTTP Routing
│   ├─ TLS Termination
│   └─ Domain Based Routing

└── CNI Plugins
    ├─ Calico
    ├─ Flannel
    ├─ Cilium
    └─ Weave

══════════════════════════════════════════════════════════════════════════════════════════════

💾 STORAGE

├── Volume
├── Persistent Volume (PV)
├── Persistent Volume Claim (PVC)
├── Storage Class
└── CSI Driver

══════════════════════════════════════════════════════════════════════════════════════════════

🔐 SECURITY

├── Authentication
│   ├─ Certificates
│   ├─ Tokens
│   └─ Service Accounts

├── Authorization
│   ├─ RBAC
│   ├─ Roles
│   ├─ ClusterRoles
│   └─ Bindings

├── Network Policy
├── Pod Security
├── Secrets
└── Security Context

══════════════════════════════════════════════════════════════════════════════════════════════

📦 WORKLOADS

├── Pod
├── ReplicaSet
├── Deployment
├── StatefulSet
├── DaemonSet
├── Job
└── CronJob

══════════════════════════════════════════════════════════════════════════════════════════════

🔄 REQUEST FLOW

User
 │
 ▼
kubectl apply -f app.yaml
 │
 ▼
API Server
 │
 ▼
etcd (store desired state)
 │
 ▼
Scheduler chooses node
 │
 ▼
Kubelet receives task
 │
 ▼
Container Runtime starts Pod
 │
 ▼
Pod Running
 │
 ▼
Service → Ingress → User Access

══════════════════════════════════════════════════════════════════════════════════════════════

🎯 INTERVIEW ONE-LINERS

✅ API Server = Front Door of Kubernetes

✅ ETCD = Brain Memory / Database

✅ Scheduler = Decides WHERE Pod Runs

✅ Controller Manager = Ensures Desired State

✅ Kubelet = Node Agent

✅ Container Runtime = Runs Containers

✅ Kube-Proxy = Service Networking

✅ Pod = Smallest Deployable Unit

✅ Service = Stable Access to Pods

✅ Ingress = HTTP/HTTPS Entry Point

✅ PV/PVC = Persistent Storage

✅ RBAC = Access Control
```

### Super-Short Memory Trick

```text
CONTROL PLANE
↓
API Server → ETCD → Scheduler → Controllers

WORKER NODE
↓
Kubelet → Runtime → Kube-Proxy → Pods

ACCESS
↓
Service → Ingress → User

STORAGE
↓
PV → PVC → StorageClass

SECURITY
↓
Auth → RBAC → Secrets → Policies
```

For CKA and 0–3 year DevOps interviews, memorize this sequence:

**API Server → ETCD → Scheduler → Controller Manager → Kubelet → Container Runtime → Pod → Service → Ingress**.

If you can explain that flow clearly, you can answer most Kubernetes architecture interview questions.
