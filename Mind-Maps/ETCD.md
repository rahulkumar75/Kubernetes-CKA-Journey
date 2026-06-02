## ETCD Mind Map (CKA + Interview Revision)

```text
                          ┌─────────────────┐
                          │      ETCD       │
                          │ K8s Database    │
                          └────────┬────────┘
                                   │
      ┌────────────────────────────┼────────────────────────────┐
      │                            │                            │
      ▼                            ▼                            ▼

┌───────────────┐        ┌────────────────┐         ┌────────────────┐
│  PURPOSE      │        │ STORES DATA    │         │ CHARACTERISTICS│
└──────┬────────┘        └───────┬────────┘         └───────┬────────┘
       │                         │                          │
       ├─ Cluster State          ├─ Pods                    ├─ Key-Value Store
       ├─ Source of Truth        ├─ Deployments             ├─ Distributed DB
       ├─ API Server Backend     ├─ Services                ├─ Highly Available
       └─ Control Plane DB       ├─ Secrets                 ├─ Consistent
                                 ├─ ConfigMaps              └─ TLS Secured
                                 ├─ Nodes
                                 └─ RBAC

                                   │
                                   ▼

                         ┌──────────────────┐
                         │  K8S FLOW        │
                         └────────┬─────────┘
                                  │

      kubectl
          │
          ▼
   kube-apiserver
          │
          ▼
        ETCD
          │
          ▼
   Cluster State Saved

──────────────────────────────────────────────────────────────

                    ┌────────────────────┐
                    │ ETCD BACKUP        │
                    └─────────┬──────────┘
                              │

                      snapshot save
                              │

ETCDCTL_API=3 etcdctl snapshot save backup.db
    --endpoints=https://127.0.0.1:2379
    --cacert=ca.crt
    --cert=server.crt
    --key=server.key

                              │
                              ▼

                      backup.db created

──────────────────────────────────────────────────────────────

                    ┌────────────────────┐
                    │ VERIFY BACKUP      │
                    └─────────┬──────────┘
                              │

ETCDCTL_API=3 etcdctl snapshot status backup.db

                              │
                              ▼

        Check HASH / REVISION / SIZE

──────────────────────────────────────────────────────────────

                    ┌────────────────────┐
                    │ ETCD RESTORE       │
                    └─────────┬──────────┘
                              │

ETCDCTL_API=3 etcdctl snapshot restore backup.db
      --data-dir=/var/lib/etcd-backup

                              │
                              ▼

         Edit etcd.yaml Manifest

                              │

Change:

--data-dir=/var/lib/etcd

To:

--data-dir=/var/lib/etcd-backup

                              │
                              ▼

             Kubelet Restarts ETCD

                              │
                              ▼

                  Cluster Recovered

──────────────────────────────────────────────────────────────

                    ┌────────────────────┐
                    │ IMPORTANT FILES    │
                    └─────────┬──────────┘
                              │

/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/server.key

Manifest:

/etc/kubernetes/manifests/etcd.yaml

──────────────────────────────────────────────────────────────

                    ┌────────────────────┐
                    │ CKA EXAM FLOW      │
                    └─────────┬──────────┘
                              │

1. SSH Control Plane
2. Find etcd.yaml
3. Locate certs
4. Snapshot Save
5. Verify Snapshot
6. Restore Snapshot
7. Change data-dir
8. Verify Cluster

──────────────────────────────────────────────────────────────

                    ┌────────────────────┐
                    │ INTERVIEW Q&A      │
                    └─────────┬──────────┘
                              │

Q. What is ETCD?
→ Distributed Key-Value Database

Q. Why is ETCD important?
→ Stores complete cluster state

Q. What happens if ETCD is down?
→ API Server cannot function properly

Q. Does ETCD store images?
→ No, only metadata/state

Q. Backup command?
→ snapshot save

Q. Restore command?
→ snapshot restore
```

### 30-Second Revision Formula

```text
ETCD = Kubernetes Brain

Backup:
snapshot save → backup.db

Verify:
snapshot status

Restore:
snapshot restore

After Restore:
Edit etcd.yaml
Change data-dir
Kubelet restarts ETCD

Files:
ca.crt
server.crt
server.key
etcd.yaml
```

For a **0–3 year DevOps/CKA interview**, this ETCD coverage is sufficient. The only advanced topics worth adding later are:

* ETCD quorum (odd number of members)
* Leader election
* Stacked vs External ETCD
* ETCD HA architecture
* Defragmentation (`etcdctl defrag`) for production clusters.
