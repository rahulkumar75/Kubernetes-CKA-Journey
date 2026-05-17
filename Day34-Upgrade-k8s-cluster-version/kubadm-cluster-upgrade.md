# Kubernetes Upgrade Guide using `kubeadm` (CKA + Real Production Style)

This is the standard workflow used to upgrade a Kubernetes cluster created with `kubeadm`.

We’ll cover:

* Control Plane upgrade
* Worker Node upgrade
* Drain / Uncordon
* kubeadm / kubelet / kubectl upgrade flow
* Real production practices
* Common mistakes
* CKA exam shortcuts

---

# 1. Understand Kubernetes Version Upgrade Rules

Example:

```bash
v1.30.x → v1.31.x
```

Supported:

* Only one minor version at a time
* Example:

  * ✅ `1.29 → 1.30`
  * ❌ `1.29 → 1.31`

---

# 2. Check Current Cluster Version

```bash
kubectl get nodes
kubectl version --short
kubeadm version
```

Example:

```bash
NAME       STATUS   VERSION
master     Ready    v1.30.2
worker1    Ready    v1.30.2
```

---

# 3. Read Upgrade Plan

This is VERY important in real production.

```bash
sudo kubeadm upgrade plan
```

This command shows:

* Current version
* Available versions
* Components needing upgrade
* CoreDNS compatibility
* etcd upgrade info

Example:

```bash
Components that must be upgraded manually after you have upgraded the control plane with 'kubeadm upgrade apply':
COMPONENT   CURRENT   TARGET
kubelet     1.30.2    1.31.0
kubectl     1.30.2    1.31.0
```

---

# 4. Upgrade Control Plane Node

---

# Step 1 — Drain Control Plane Node

```bash
kubectl drain master --ignore-daemonsets
```

If single-node cluster:

```bash
kubectl drain master --ignore-daemonsets --delete-emptydir-data
```

Purpose:

* Safely evicts workloads
* Prevents scheduling during upgrade

---

# Step 2 — Upgrade kubeadm

Ubuntu/Debian example:

```bash
sudo apt update
sudo apt-cache madison kubeadm
```

Install target version:

```bash
sudo apt install -y kubeadm=1.31.0-1.1
```

Check:

```bash
kubeadm version
```

---

# Step 3 — Apply Upgrade

```bash
sudo kubeadm upgrade apply v1.31.0
```

This upgrades:

* API Server
* Controller Manager
* Scheduler
* etcd
* CoreDNS (if needed)

---

# Step 4 — Upgrade kubelet + kubectl

```bash
sudo apt install -y kubelet=1.31.0-1.1 kubectl=1.31.0-1.1
```

Restart kubelet:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

# Step 5 — Uncordon Node

```bash
kubectl uncordon master
```

---

# 5. Upgrade Worker Nodes

Repeat one-by-one.

---

# Step 1 — Drain Worker Node

```bash
kubectl drain worker1 --ignore-daemonsets
```

---

# Step 2 — Upgrade kubeadm

```bash
sudo apt install -y kubeadm=1.31.0-1.1
```

---

# Step 3 — Upgrade Node

Worker nodes use:

```bash
sudo kubeadm upgrade node
```

NOT:

```bash
kubeadm upgrade apply
```

Difference:

| Command                 | Used On       |
| ----------------------- | ------------- |
| `kubeadm upgrade apply` | Control Plane |
| `kubeadm upgrade node`  | Worker Node   |

---

# Step 4 — Upgrade kubelet + kubectl

```bash
sudo apt install -y kubelet=1.31.0-1.1 kubectl=1.31.0-1.1
```

Restart:

```bash
sudo systemctl daemon-reload
sudo systemctl restart kubelet
```

---

# Step 5 — Uncordon Worker

```bash
kubectl uncordon worker1
```

---

# 6. Verify Upgrade

```bash
kubectl get nodes
```

Expected:

```bash
NAME       STATUS   VERSION
master     Ready    v1.31.0
worker1    Ready    v1.31.0
```

Check pods:

```bash
kubectl get pods -A
```

---

# REAL PRODUCTION UPGRADE FLOW

Real companies usually do:

```text
1. Take etcd backup
2. Read release notes
3. Upgrade staging cluster
4. Upgrade control plane
5. Upgrade workers one-by-one
6. Monitor apps
7. Rollback if needed
```

---

# Important Production Concepts

## Why Drain Node?

During upgrade:

* kubelet restarts
* containers restart
* workloads may fail

Drain safely moves pods elsewhere.

---

# Why Worker Nodes One-by-One?

To avoid downtime.

If all workers upgraded together:

* apps become unavailable
* traffic fails

---

# Why kubeadm First?

Because kubeadm manages cluster component versions.

Correct order:

```text
kubeadm
↓
control plane
↓
kubelet
↓
kubectl
```

---

# Important Commands Summary

| Purpose               | Command                                  |
| --------------------- | ---------------------------------------- |
| Check versions        | `kubectl get nodes`                      |
| View upgrade plan     | `kubeadm upgrade plan`                   |
| Drain node            | `kubectl drain NODE --ignore-daemonsets` |
| Upgrade control plane | `kubeadm upgrade apply v1.xx.x`          |
| Upgrade worker        | `kubeadm upgrade node`                   |
| Restart kubelet       | `systemctl restart kubelet`              |
| Enable scheduling     | `kubectl uncordon NODE`                  |

---

# Common Upgrade Failures

---

## 1. Forgot Drain

Symptoms:

* App downtime
* Restart storms

---

## 2. Version Skipped

Example:

```text
1.28 → 1.30
```

Fails because only one minor version supported.

---

## 3. kubelet Older/Newer Mismatch

Check:

```bash
kubectl get nodes
```

Version mismatch may cause:

* Node NotReady
* kubelet registration issues

---

## 4. PDB Blocking Drain

Example:

```bash
cannot evict pod as it would violate PodDisruptionBudget
```

Fix:

```bash
kubectl drain NODE --ignore-daemonsets --force
```

or temporarily modify PDB.

---

# CKA Exam Important Notes

Very frequently asked.

Remember:

```text
Control Plane:
kubeadm upgrade apply

Worker:
kubeadm upgrade node
```

---

# Fast CKA Cheat Sheet

## Control Plane

```bash
kubectl drain master --ignore-daemonsets

apt install kubeadm=VERSION

kubeadm upgrade apply vVERSION

apt install kubelet=VERSION kubectl=VERSION

systemctl restart kubelet

kubectl uncordon master
```

---

## Worker

```bash
kubectl drain worker1 --ignore-daemonsets

apt install kubeadm=VERSION

kubeadm upgrade node

apt install kubelet=VERSION kubectl=VERSION

systemctl restart kubelet

kubectl uncordon worker1
```

---

# Interview Questions

## Difference between:

### `kubeadm upgrade apply`

vs

### `kubeadm upgrade node`

| apply                      | node                 |
| -------------------------- | -------------------- |
| Upgrades control plane     | Upgrades worker node |
| Updates cluster components | Updates node config  |
| Used on master             | Used on workers      |

---

## Why drain before upgrade?

To safely evict workloads and avoid downtime.

---

## What components get upgraded?

* kube-apiserver
* controller-manager
* scheduler
* etcd
* CoreDNS
* kube-proxy

---

## What if upgrade fails?

* Check kubelet logs
* Rollback using snapshots/backups
* Restore etcd backup

---

# etcd Backup Before Upgrade (VERY IMPORTANT)

Take backup:

```bash
ETCDCTL_API=3 etcdctl snapshot save backup.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

Verify:

```bash
etcdctl snapshot status backup.db
```

---

# Visual Upgrade Flow

```text
Control Plane Upgrade
---------------------
Drain Node
   ↓
Upgrade kubeadm
   ↓
kubeadm upgrade apply
   ↓
Upgrade kubelet/kubectl
   ↓
Restart kubelet
   ↓
Uncordon Node

Worker Upgrade
---------------
Drain Node
   ↓
Upgrade kubeadm
   ↓
kubeadm upgrade node
   ↓
Upgrade kubelet/kubectl
   ↓
Restart kubelet
   ↓
Uncordon Node
```
