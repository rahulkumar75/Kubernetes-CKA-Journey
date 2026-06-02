# Real-Time Production Guide: PV, PVC & StorageClass (Hands-on)

Now moving from **CKA exam mode** to **real DevOps production mode**.

In production, we usually **do not use hostPath**.
We use CSI-based storage like:

* AWS EBS / EFS
* Azure Disk / Files
* GCP Persistent Disk
* Ceph / Longhorn / Portworx
* NetApp / SAN / NAS

---

# 🧠 Real Production Architecture

```text id="pp0q2l"
Application Pod
    ↓
PVC (request storage)
    ↓
StorageClass (rules/template)
    ↓
CSI Driver provisions disk
    ↓
PV created automatically
    ↓
Disk attached to node
```

---

# Real Use Cases

## Databases

* MySQL
* PostgreSQL
* MongoDB
* Redis (AOF/RDB persistence)

## CI/CD

* Jenkins home directory

## Logging

* Elasticsearch/OpenSearch

## Shared Media

* uploads / user files

---

# Example Production Setup (AWS EKS)

We’ll simulate a real-world PostgreSQL deployment using:

* StorageClass = gp3
* PVC = 20Gi
* Pod = postgres

---

# Step 1️⃣ Check StorageClass

```bash id="pr01"
kubectl get sc
```

Example output:

```text id="pr02"
gp2
gp3 (default)
efs-sc
```

---

# Step 2️⃣ Create PVC for Database

```yaml id="pr03"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: gp3
  resources:
    requests:
      storage: 20Gi
```

Apply:

```bash id="pr04"
kubectl apply -f pvc.yaml
```

---

# What Happens Internally

```text id="pr05"
PVC created
↓
EBS CSI driver called
↓
20Gi EBS volume created
↓
PV auto-created
↓
PVC bound
```

---

# Step 3️⃣ Verify

```bash id="pr06"
kubectl get pvc
kubectl get pv
```

Expected:

```text id="pr07"
STATUS = Bound
```

---

# Step 4️⃣ Deploy PostgreSQL with PVC

```yaml id="pr08"
apiVersion: v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15
        env:
        - name: POSTGRES_PASSWORD
          value: admin123
        volumeMounts:
        - mountPath: /var/lib/postgresql/data
          name: db-storage
      volumes:
      - name: db-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
```

Apply:

```bash id="pr09"
kubectl apply -f postgres.yaml
```

---

# Step 5️⃣ Validate Persistence

Create data:

```bash id="pr10"
kubectl exec -it deploy/postgres -- bash
```

Inside:

```bash id="pr11"
touch /var/lib/postgresql/data/test.txt
exit
```

Restart pod:

```bash id="pr12"
kubectl delete pod -l app=postgres
```

Check again:

```bash id="pr13"
kubectl exec -it deploy/postgres -- ls /var/lib/postgresql/data
```

✅ file still exists

---

# 🔥 Real Production Scenario: Node Failure

If node crashes:

```text id="pr14"
Pod rescheduled to another node
↓
EBS volume detached old node
↓
Attached to new node
↓
Pod starts again
```

✅ data preserved

---

# ReadWriteOnce vs ReadWriteMany

## EBS (RWO)

One node at a time.

Use for:

* PostgreSQL
* MySQL
* MongoDB

## EFS / NFS (RWX)

Many pods / many nodes.

Use for:

* shared uploads
* Jenkins shared home
* WordPress media

---

# Example RWX Production (EFS)

```yaml id="pr15"
storageClassName: efs-sc
accessModes:
- ReadWriteMany
```

Multiple pods can mount same PVC.

---

# Production Best Practices

## 1️⃣ Separate App & Data

```text id="pr16"
Deployment for app
PVC for data
```

Never store DB inside container layer.

---

## 2️⃣ Use StatefulSet for DB

For databases prefer:

```text id="pr17"
StatefulSet + volumeClaimTemplates
```

Instead of normal Deployment.

---

## 3️⃣ Backups

PVC ≠ backup

Need:

* EBS snapshots
* Velero
* Restic
* DB native backup

---

## 4️⃣ Encryption

Use encrypted volumes.

AWS:

```text id="pr18"
EBS encrypted with KMS
```

---

## 5️⃣ Monitor Capacity

```bash id="pr19"
kubectl describe pvc
```

Also Prometheus alerts.

---

# Real Production Troubleshooting

---

## PVC Pending

```bash id="pr20"
kubectl describe pvc postgres-pvc
```

Possible:

* no StorageClass
* CSI driver missing
* quota exceeded
* IAM issue

---

## Pod Pending

Volume attach issue:

```bash id="pr21"
kubectl describe pod <pod>
```

---

## Multi-Attach Error

If RWO disk already attached elsewhere.

Common with EBS.

---

# Real DevOps Interview Answer

**In production we create PVC using a StorageClass. Kubernetes uses CSI driver to dynamically provision storage like EBS/EFS. PVC is mounted to stateful workloads such as PostgreSQL, Jenkins, Elasticsearch. If pod restarts or moves nodes, storage is reattached and data persists.**

---

# Hands-on for KIND (Production-like)

Use local-path:

```bash id="pr22"
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

Then:

```yaml id="pr23"
storageClassName: local-path
```

Good for learning.

---

# Production Comparison

| Environment | StorageClass | Use                 |
| ----------- | ------------ | ------------------- |
| EKS         | gp3          | DB volumes          |
| EKS         | efs-sc       | Shared storage      |
| AKS         | managed-csi  | Azure disks         |
| GKE         | standard-rwo | PD volumes          |
| On-prem     | rook-ceph    | Distributed storage |

---

# If I Were Building Your Real MERN App on Kubernetes

## MongoDB

PVC 30Gi gp3

## Backend

Stateless Deployment

## Frontend

Stateless Deployment

## User uploads

RWX EFS PVC

---

