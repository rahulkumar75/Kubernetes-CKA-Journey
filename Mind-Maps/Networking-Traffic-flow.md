# 🗺️ Kubernetes Networking & Traffic Flow Mind Map (0–3 Years Interview Revision)

```text
KUBERNETES NETWORKING & TRAFFIC FLOW
│
├── 1. Core Networking Principles
│   │
│   ├── Every Pod gets its own IP
│   ├── Pods can communicate without NAT
│   ├── Flat Network Model
│   ├── No Port Mapping between Pods
│   └── Pod IPs are routable
│
├── 2. CoreDNS (Service Discovery)
│   │
│   ├── Purpose
│   │   ├── Name Resolution
│   │   └── Service Discovery
│   │
│   ├── Example
│   │   └── backend-svc → 10.96.0.10
│   │
│   ├── Traffic Flow
│   │   ├── Pod → DNS Query
│   │   ├── CoreDNS
│   │   └── Returns Service IP
│   │
│   └── Troubleshooting
│       ├── nslookup service-name
│       ├── kubectl get pods -n kube-system
│       └── Check CoreDNS logs
│
├── 3. Service (ClusterIP)
│   │
│   ├── Stable Virtual IP
│   ├── Load Balancing
│   ├── Decouples Pods
│   └── Types
│       ├── ClusterIP
│       ├── NodePort
│       ├── LoadBalancer
│       └── ExternalName
│
├── 4. kube-proxy
│   │
│   ├── Watches
│   │   ├── Services
│   │   └── Endpoints
│   │
│   ├── Creates
│   │   ├── iptables Rules
│   │   └── IPVS Rules
│   │
│   ├── Responsibilities
│   │   ├── Service Routing
│   │   ├── Load Balancing
│   │   └── DNAT
│   │
│   └── Example
│       └── ClusterIP → Backend Pod IP
│
├── 5. CNI (Container Network Interface)
│   │
│   ├── Purpose
│   │   ├── Assign Pod IP
│   │   ├── Configure Networking
│   │   └── Route Traffic
│   │
│   ├── Popular CNIs
│   │   ├── Calico
│   │   ├── Flannel
│   │   ├── Cilium
│   │   └── Weave
│   │
│   ├── Same Node Communication
│   │   ├── veth Pair
│   │   ├── Linux Bridge
│   │   └── Direct Forwarding
│   │
│   └── Different Node Communication
│       ├── Overlay Network
│       ├── VXLAN
│       └── Routing/BGP
│
├── 6. NAT in Kubernetes
│   │
│   ├── DNAT
│   │   ├── Service IP
│   │   └── Pod IP
│   │
│   ├── SNAT
│   │   ├── Pod IP
│   │   └── Node IP
│   │
│   └── Used For
│       ├── Service Routing
│       └── External Communication
│
├── 7. Pod-to-Pod Traffic Flow
│   │
│   ├── Same Node
│   │   │
│   │   ├── Pod A
│   │   ├── veth
│   │   ├── Linux Bridge
│   │   └── Pod B
│   │
│   └── Different Nodes
│       │
│       ├── Pod A
│       ├── Node 1
│       ├── CNI Network
│       ├── Node 2
│       └── Pod B
│
├── 8. Pod-to-Service Traffic Flow
│   │
│   ├── Step 1: Pod requests Service
│   ├── Step 2: CoreDNS resolves name
│   ├── Step 3: Service IP returned
│   ├── Step 4: kube-proxy DNAT
│   ├── Step 5: CNI routes packet
│   └── Step 6: Backend Pod receives packet
│
├── 9. Packet Debugging Tools
│   │
│   ├── tcpdump
│   │   └── Capture Actual Packets
│   │
│   ├── nslookup
│   │   └── DNS Verification
│   │
│   ├── curl
│   │   └── Connectivity Test
│   │
│   ├── ip route
│   │   └── Routing Table
│   │
│   ├── iptables
│   │   └── NAT Rules
│   │
│   └── kubectl exec
│       └── Run Commands in Pod
│
├── 10. Common Failures
│   │
│   ├── DNS Failure
│   │   └── CoreDNS Down
│   │
│   ├── Service Failure
│   │   └── No Endpoints
│   │
│   ├── kube-proxy Failure
│   │   └── Missing DNAT Rules
│   │
│   ├── CNI Failure
│   │   └── Pod Routing Broken
│   │
│   ├── NetworkPolicy Block
│   │   └── Traffic Denied
│   │
│   └── NAT Issue
│       └── External Access Fails
│
└── 11. Interview Golden Flow
    │
    ├── DNS (CoreDNS)
    ├── Service (ClusterIP)
    ├── kube-proxy
    ├── Endpoints
    ├── CNI
    ├── NAT
    ├── NetworkPolicy
    └── tcpdump Verification
```

---

# 🎯 30-Second Interview Answer

> When a pod accesses a service, it first resolves the service name through CoreDNS. CoreDNS returns the Service ClusterIP. The packet reaches kube-proxy, which performs DNAT and forwards traffic to one of the backend pods. The CNI plugin handles actual routing between nodes. If traffic leaves the cluster, SNAT may occur. For troubleshooting, I verify DNS, Service endpoints, kube-proxy rules, CNI routing, NetworkPolicies, and finally inspect packets using tcpdump.

## Must-Remember Keywords (0–3 Years)

```text
CoreDNS      → Name Resolution
Service      → Stable Virtual IP
kube-proxy   → DNAT + Load Balancing
CNI          → Pod Networking
DNAT         → Service IP → Pod IP
SNAT         → Pod IP → Node IP
tcpdump      → Packet Capture
NetworkPolicy→ Traffic Control
Endpoints    → Actual Backend Pods
```

This single mind map covers nearly all Kubernetes networking questions typically asked in CKA and 0–3 year DevOps/Kubernetes interviews.
