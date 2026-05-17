# Complete Guide on ETCD Backup in Kubernetes

In Kubernetes, **etcd** is the brain/database of the cluster.

It stores:

* Cluster state
* Nodes
* Pods
* Deployments
* Secrets
* ConfigMaps
* RBAC
* Everything created using `kubectl`

If etcd is lost/corrupted:

* Cluster state is gone
* Kubernetes API becomes unusable
* Recovery becomes very difficult without backup

---

# 1. Architecture Understanding

Control Plane Components:

```text
kubectl
   ↓
kube-apiserver
   ↓
etcd
```

`kube-apiserver` reads/writes all cluster data into etcd.

---

# 2. Check ETCD is Running

Usually on control-plane node:

```bash
kubectl get pods -n kube-system
```

Find:

```bash
etcd-controlplane
```

OR check process:

```bash
ps -ef | grep etcd
```

---

# 3. Important ETCD Files

Most important certificates:

```bash
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/server.key
```

---

# 4. Verify ETCD Endpoint

Check manifest:

```bash
cat /etc/kubernetes/manifests/etcd.yaml
```

Look for:

```yaml
--listen-client-urls=https://127.0.0.1:2379
```

Default endpoint:

```bash
https://127.0.0.1:2379
```

---

# 5. Take ETCD Backup (Most Important)

Use:

```bash
ETCDCTL_API=3 etcdctl snapshot save backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

---

# 6. Verify Backup

```bash
ETCDCTL_API=3 etcdctl snapshot status backup.db
```

Example output:

```text
HASH      REVISION  TOTAL KEYS  TOTAL SIZE
xxxx      12345     900         25 MB
```

---

# 7. Backup Flow (Real Understanding)

```text
etcdctl
   ↓
Connects securely using TLS certs
   ↓
Reads full etcd database snapshot
   ↓
Stores snapshot as backup.db
```

---

# 8. Restore ETCD Backup

Suppose cluster is broken.

Restore using:

```bash
ETCDCTL_API=3 etcdctl snapshot restore backup.db \
  --data-dir=/var/lib/etcd-backup
```

This restores snapshot into new data directory.

---

# 9. Update ETCD Manifest After Restore

Edit:

```bash
vi /etc/kubernetes/manifests/etcd.yaml
```

Change:

```yaml
--data-dir=/var/lib/etcd
```

TO:

```yaml
--data-dir=/var/lib/etcd-backup
```

Save file.

Kubelet automatically restarts etcd static pod.

---

# 10. Verify Cluster

Check:

```bash
kubectl get nodes
kubectl get pods -A
```

---

# 11. Important CKA Exam Commands

## Backup

```bash
ETCDCTL_API=3 etcdctl snapshot save backup.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=<ca.crt> \
--cert=<server.crt> \
--key=<server.key>
```

---

## Restore

```bash
ETCDCTL_API=3 etcdctl snapshot restore backup.db \
--data-dir=/var/lib/etcd-backup
```

---

# 12. Real Production Best Practices

## Take Scheduled Backups

Usually:

* Every 30 mins
* Hourly
* Daily retention

Using:

* CronJobs
* Velero
* Managed backup systems

---

## Store Backup Outside Node

Never keep only local copy.

Store in:

* S3
* NAS
* Backup server
* Object storage

---

## Encrypt Backups

Because etcd contains:

* Secrets
* Tokens
* Certificates

---

## Verify Restore Regularly

Backup without restore testing = dangerous.

Always test restoration.

---

# 13. Common Failure Scenarios

## Scenario 1: Accidentally Deleted Namespace

Restore etcd snapshot.

---

## Scenario 2: Control Plane Disk Corruption

Restore etcd on new node.

---

## Scenario 3: Ransomware / Human Mistake

Recover full cluster state from snapshot.

---

# 14. Troubleshooting

## Error: connection refused

Check etcd running:

```bash
crictl ps | grep etcd
```

OR:

```bash
docker ps | grep etcd
```

---

## Error: certificate issue

Verify correct:

* ca.crt
* server.crt
* server.key

---

## Error: cluster not coming up after restore

Usually:

* wrong `data-dir`
* typo in manifest
* old corrupted data still exists

---

# 15. Fast CKA Exam Method

## Find certificates quickly

```bash
cat /etc/kubernetes/manifests/etcd.yaml | grep crt
```

---

## Find key quickly

```bash
cat /etc/kubernetes/manifests/etcd.yaml | grep key
```

---

# 16. One-Line Memory Trick

```text
Backup:
snapshot save

Restore:
snapshot restore

Then:
change data-dir in etcd.yaml
```

---

# 17. Most Important Interview Questions

## Q1. Why is etcd important?

Because it stores complete Kubernetes cluster state.

---

## Q2. Does etcd store container images?

No.

It stores metadata/state only.

---

## Q3. What happens if etcd goes down?

Kubernetes API server becomes unusable.

Cluster state operations fail.

---

## Q4. Which protocol does etcd use?

gRPC over TLS.

---

## Q5. Difference between backup and restore?

* Backup → take snapshot
* Restore → rebuild cluster state from snapshot

---

# 18. Real Exam Workflow

```text
1. SSH into control-plane
2. Find etcd cert paths
3. Take snapshot
4. Verify snapshot
5. Restore if required
6. Change data-dir
7. Wait for kubelet restart
8. Verify cluster
```

---

# 19. Visual Revision

```text
ETCD BACKUP:

etcdctl snapshot save backup.db

ETCD RESTORE:

etcdctl snapshot restore backup.db \
--data-dir=/new/location

THEN:

Edit etcd.yaml
Change --data-dir
Kubelet restarts etcd
Cluster restored
```
