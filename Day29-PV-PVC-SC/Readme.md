## Kubernetes Volume | Persistent Volume (PV), Persistent Volume Claim (PVC) & StorageClass

(Interview + Hands-on + Real-world + CKA Style)

This topic is **very important** for CKA and real-world Kubernetes because containers are **ephemeral**.

If container restarts → data inside container is lost.
So Kubernetes uses **Volumes** for persistent storage.

---

# 1️⃣ First Understand the Problem

Imagine MySQL Pod:

```yaml
Pod:
  mysql container
```

If pod deleted/restarted:

❌ DB files lost

Need storage outside container lifecycle.

That is why:

✅ Volumes
✅ PersistentVolume
✅ PersistentVolumeClaim
✅ StorageClass

---

# 2️⃣ Kubernetes Volume Types (Basic)

### EmptyDir

Temporary storage. Lives till Pod lives.

```yaml
volumes:
- name: cache
  emptyDir: {}
```

Use case:

* cache
* temp files
* sidecar logs

---

### HostPath

Mount node directory into pod.

```yaml
hostPath:
  path: /data
```

Use for lab/testing.

Not ideal production.

---

# 3️⃣ Persistent Storage Flow (Most Important)

## Real Meaning

### PersistentVolume (PV)

Storage resource created in cluster.

Example:

* 10Gi disk
* NFS share
* AWS EBS volume
* Azure Disk

Think:

👉 Admin provides storage.

---

### PersistentVolumeClaim (PVC)

Developer requests storage.

Example:

* Need 5Gi
* Need ReadWriteOnce

Think:

👉 User asks for storage.

---

### StorageClass

Blueprint for dynamic provisioning.

Example:

* gp2 (AWS EBS SSD)
* fast-ssd
* slow-hdd

Think:

👉 Auto-create storage when PVC requested.

---

# 4️⃣ Visual Flow

```text
Developer creates PVC
        ↓
StorageClass dynamically creates PV
        ↓
PVC binds PV
        ↓
Pod mounts PVC
```

---

# 5️⃣ Real Example (Static Provisioning)

## Step 1: Create PV

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /mnt/data
```

Apply:

```bash
kubectl apply -f pv.yaml
```

---

## Step 2: Create PVC

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 2Gi
```

Apply:

```bash
kubectl apply -f pvc.yaml
```

Check:

```bash
kubectl get pv,pvc
```

PVC should bind to PV.

---

## Step 3: Use PVC in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: myvol

  volumes:
  - name: myvol
    persistentVolumeClaim:
      claimName: pvc-demo
```

---

# 6️⃣ Dynamic Provisioning (Modern Production)

Instead of manually creating PV.

Use StorageClass.

## Create PVC only:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: auto-pvc
spec:
  storageClassName: gp2
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

Then Kubernetes auto-creates PV.

---

# 7️⃣ Access Modes (Interview Important)

### ReadWriteOnce (RWO)

One node can mount read-write.

Most common (EBS).

### ReadOnlyMany (ROX)

Many nodes read-only.

### ReadWriteMany (RWX)

Many nodes read-write.

Used with:

* NFS
* EFS
* CephFS

---

# 8️⃣ Reclaim Policy

What happens after PVC deleted?

### Retain

Keep data manually.

### Delete

Delete disk automatically.

### Recycle (deprecated)

---

# 9️⃣ Real-Time Use Cases

## Database

MySQL / PostgreSQL needs persistent data.

## Jenkins

Need job configs + workspace.

## Elasticsearch

Stores indices.

## Shared files

RWX volume.

---

# 🔟 Important Commands (CKA)

```bash
kubectl get pv
kubectl get pvc
kubectl describe pv pv-demo
kubectl describe pvc pvc-demo
kubectl get storageclass
kubectl get sc
```

---

# 1️⃣1️⃣ Troubleshooting

## PVC Pending

Reasons:

* No matching PV
* Wrong storageClass
* insufficient size
* wrong accessMode

Check:

```bash
kubectl describe pvc pvc-demo
```

---

## Pod Pending

PVC not bound.

---

# 1️⃣2️⃣ Interview Answer (Best)

**What is PV and PVC?**

PV is actual storage resource in cluster.
PVC is request for storage by user.
PVC binds to suitable PV and pod mounts PVC.

---

# 1️⃣3️⃣ CKA Fast Imperative Practice

Create PVC quickly:

```bash
kubectl create pvc myclaim --access-mode=ReadWriteOnce --request=1Gi
```

(If supported version)

---

# 1️⃣4️⃣ Real Production Advice

For EKS:

* EBS CSI Driver
* EFS CSI Driver

For AKS:

* Azure Disk / Azure Files

For GKE:

* Persistent Disk / Filestore

---

# 1️⃣5️⃣ Easy Memory Trick

```text
PV  = Provided Volume
PVC = Volume Request Ticket
SC  = Auto Volume Factory
```

---

# 1️⃣6️⃣ If CKA asks:

> Create PVC and mount in pod

Steps:

```bash
kubectl create pvc ...
kubectl run pod ...
edit pod yaml add volumeMount + pvc
```

---


