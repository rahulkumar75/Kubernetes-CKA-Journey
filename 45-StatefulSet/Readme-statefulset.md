# Kubernetes StatefulSets: A Complete Guide

## Overview

Kubernetes manages two fundamentally different categories of applications, each with its own resource type:

| Type | Resources | Use Case |
|------|-----------|----------|
| **Stateless** | Deployment, ReplicaSet | Applications that do not need to persist data between restarts |
| **Stateful** | StatefulSet | Applications that require persistent storage, dedicated databases, or a stable identity |

---

## Stateless vs. Stateful: Key Differences

### Stateless Applications

- Pod names are **randomly generated** and unpredictable.
- Pods can be created, deleted, or replaced **in any order**, even in parallel.
- There is **no guaranteed network identity** — any pod can serve any request.

**Example resources:** `Deployment`, `ReplicaSet`

---

### Stateful Applications

- Each pod gets a **sequentially assigned name** (e.g., `mongodb-0`, `mongodb-1`, `mongodb-2`).
- Pods are created and deleted **one at a time, in order** — following their ordinal index.
- Each pod has a **stable, predictable network identity** that persists across restarts.
- Each pod gets its **own dedicated PersistentVolumeClaim (PVC)**, ensuring data survives pod restarts.

**Key guarantee:** Even if a pod crashes and is recreated, it will reattach to the same PVC, preserving all its data.

---

## Architecture: What You'll Build

This guide walks through deploying a **MongoDB StatefulSet** with 3 replicas, backed by local persistent storage. The setup involves five components:

```
Headless Service  →  StatefulSet (3 pods)
                           ↓
                   PVC per pod (1:1 mapping)
                           ↓
                   PersistentVolume (PV)
                           ↓
                   StorageClass (no-provisioner)
                           ↓
                   Local directories on Worker Node
```

---

## Step-by-Step Implementation

### Step 1: Prepare Directories on the Worker Node

On the **worker node** (not the control plane), create the five directories that will serve as local storage volumes:

```bash
mkdir -p /data/mongo-{0,1,2,3,4}
```

These five directories map 1:1 to each pod's PersistentVolume.

---

### Step 2: Create a Headless Service

A StatefulSet **requires** a Headless Service to manage the network identity of each pod.

> **What is a Headless Service?**
> A service with `clusterIP: None`. It does not get a cluster IP or load balance traffic. Instead, it creates DNS entries for each individual pod (e.g., `mongodb-0.mongodb-service`), enabling direct pod-to-pod addressing.

Create `mongo-svc.yaml` on the **control plane**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  labels:
    app: mongodb
spec:
  clusterIP: None          # This makes it a Headless Service
  selector:
    app: mongodb
  ports:
    - port: 27017
      targetPort: 27017
```

Apply it:

```bash
kubectl apply -f mongo-svc.yaml
```

Verify — the service will show `None` for CLUSTER-IP:

```bash
kubectl get svc mongodb-service
# NAME              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
# mongodb-service   ClusterIP   None         <none>        27017/TCP   5s
```

> **Note:** The absence of an IP is expected and correct for a headless service.

---

### Step 3: Create a StorageClass

The StorageClass tells Kubernetes *how* to provision storage.

Create `mongodb-sc.yaml`:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: mongodb-sc
provisioner: kubernetes.io/no-provisioner   # Static provisioning
volumeBindingMode: WaitForFirstConsumer      # Bind only when a pod is scheduled
```

**Key settings explained:**

| Field | Value | Reason |
|-------|-------|--------|
| `provisioner` | `no-provisioner` | For local/sandbox environments; use cloud-specific provisioners (EBS, GCP Persistent Disk) on managed clusters |
| `volumeBindingMode` | `WaitForFirstConsumer` | Prevents PVs from binding before pods are scheduled; ensures locality |

Apply and verify:

```bash
kubectl apply -f mongodb-sc.yaml
kubectl get sc
```

---

### Step 4: Create PersistentVolumes (PVs)

Since we're using static provisioning, you must create a PV for each pod manually. Create `pv.yaml` with **5 PV definitions** (one per pod):

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mongodb-pv-0
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: mongodb-sc
  local:
    path: /data/mongo-0
  nodeAffinity:
    required:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - <your-worker-node-name>   # Replace with actual hostname
---
# Repeat for mongodb-pv-1 through mongodb-pv-4,
# incrementing the name and path accordingly.
```

> **Important:** There is a strict **1:1:1 mapping** between pod → PV → PVC. With 3 replicas, 3 PVs will be bound immediately; the remaining 2 stay available for future scaling.

Apply and verify:

```bash
kubectl apply -f pv.yaml
kubectl get pv
# All 5 PVs should show STATUS: Available
```

---

### Step 5: Create the StatefulSet

Create `ss.yaml`:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: "mongodb-service"   # Must match the Headless Service name
  replicas: 3
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
        - name: mongodb
          image: mongo:latest
          ports:
            - containerPort: 27017  # Must match the service port in mongo-svc.yaml
          volumeMounts:
            - name: mongodb-storage
              mountPath: /data/db
  volumeClaimTemplates:
    - metadata:
        name: mongodb-storage
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: mongodb-sc
        resources:
          requests:
            storage: 1Gi
```

**Key notes:**

- `serviceName` **must** reference your Headless Service.
- `volumeClaimTemplates` automatically creates a unique PVC for each pod (e.g., `mongodb-storage-mongodb-0`).
- The `containerPort` must match the port defined in `mongo-svc.yaml`.
- Unlike Deployments, StatefulSets **cannot** be created without a backing service.

Apply it:

```bash
kubectl apply -f ss.yaml
```

Watch pods come up **sequentially** (one at a time):

```bash
kubectl get pods -w
# mongodb-0   Running
# mongodb-1   Running   ← Only starts after mongodb-0 is Running
# mongodb-2   Running   ← Only starts after mongodb-1 is Running
```

After 3 replicas are running, verify PVC binding:

```bash
kubectl get pvc
# 3 PVCs will show STATUS: Bound
# 2 PVs remain Available for future scale-up
```

> **Note:** PVs are randomly matched to pods — Kubernetes selects from available PVs in your StorageClass.

---

## Verifying Persistence: The Crash Test

### 1. Connect to a Pod and Insert Data

```bash
kubectl exec -it mongodb-0 -- mongosh
```

In the Mongo shell:

```javascript
use testdb
db.users.insertOne({ name: "Alice", role: "admin" })
db.users.find()
// { _id: ..., name: 'Alice', role: 'admin' }
```

### 2. Delete the Pod

```bash
kubectl delete pod mongodb-0
```

### 3. Verify the Pod Recovers with Its Data

```bash
kubectl get pods -w
# mongodb-0 is recreated with the SAME name

kubectl exec -it mongodb-0 -- mongosh
use testdb
db.users.find()
# { _id: ..., name: 'Alice', role: 'admin' }  ← Data is intact ✓
```

**Why does this work?** When `mongodb-0` is recreated, it carries the same identity (`mongodb-0`) and Kubernetes reattaches its original PVC (`mongodb-storage-mongodb-0`). The data on the underlying PV is untouched.

---

## Scaling the StatefulSet

### Scale Up (3 → 5 replicas)

```bash
kubectl scale statefulset mongodb --replicas=5
```

- Pods `mongodb-3` and `mongodb-4` are created **sequentially**.
- They each claim one of the two remaining available PVs.
- Their data directories (`/data/mongo-3`, `/data/mongo-4`) on the worker node become active.

### Scale Down (5 → 3 replicas)

```bash
kubectl scale statefulset mongodb --replicas=3
```

- Pods `mongodb-4` and `mongodb-3` are deleted **in reverse order**.
- Their PVCs are **retained** (not deleted), so data is preserved if you scale back up.

Verify the worker node directories after scaling up:

```bash
ls /data/
# mongo-0  mongo-1  mongo-2  mongo-3  mongo-4
# Previously empty directories now contain MongoDB data files
```

---

## Quick Reference: StatefulSet vs. Deployment

| Feature | Deployment | StatefulSet |
|---------|------------|-------------|
| Pod naming | Random (e.g., `app-7d4f8b`) | Sequential (e.g., `app-0`, `app-1`) |
| Pod creation order | Parallel | Sequential (ordinal) |
| Pod deletion order | Random | Reverse sequential |
| Network identity | Ephemeral | Stable and predictable |
| Storage | Shared or none | Dedicated PVC per pod |
| Requires a service | No | Yes — headless service required |
| Use case | Web servers, APIs | Databases, message queues, caches |

---

## Common Pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| PVCs stay `Pending` | `volumeBindingMode` is not `WaitForFirstConsumer`, or no matching PV exists | Ensure PVs are created with the correct `storageClassName` and node affinity |
| Pods not starting sequentially | Checking too early; StatefulSets always sequence | Use `kubectl get pods -w` to observe the controlled rollout |
| Data lost after pod restart | Using a Deployment instead of a StatefulSet, or missing PVC | Use StatefulSet with `volumeClaimTemplates` |
| Service has a Cluster IP | `clusterIP: None` missing in the service spec | Add `clusterIP: None` to make it headless |
| StatefulSet won't create | `serviceName` doesn't match an existing service | Create the headless service **before** the StatefulSet |