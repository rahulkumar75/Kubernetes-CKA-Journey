## 🌳 Kubernetes ReplicaSet & Deployment - Mind Map (CKA + Interview Revision)

```text
                         REPLICASET & DEPLOYMENT
                                   │
              ┌────────────────────┴────────────────────┐
              │                                         │
              ▼                                         ▼
         REPLICASET                               DEPLOYMENT
              │                                         │
              │                                         │
      Ensures desired Pods                  Manages ReplicaSets
      are always running                    + Rolling Updates
              │                                         │
              │                                         │
      Pod deleted?                           New version?
      Creates new Pod                        Creates new RS
              │                                         │
              └──────────────┬──────────────────────────┘
                             │
                             ▼
                          PODS

═══════════════════════════════════════════════════════════════

                      OBJECT HIERARCHY

Deployment
     │
     ▼
ReplicaSet
     │
     ▼
Pods

Example:

nginx-deploy
      │
      ▼
nginx-rs-7f9b6d4
      │
      ├── Pod-1
      ├── Pod-2
      └── Pod-3

═══════════════════════════════════════════════════════════════

                     REPLICASET COMPONENTS

ReplicaSet
│
├── replicas
│     └── Desired Pod Count
│
├── selector
│     └── matchLabels
│
└── template
      └── Pod Blueprint

Example:

spec:
  replicas: 3

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

═══════════════════════════════════════════════════════════════

                     DEPLOYMENT FEATURES

Deployment
│
├── Create ReplicaSet
├── Rolling Update
├── Rollback
├── Revision History
├── Scale Up/Down
└── Zero Downtime Deployment

═══════════════════════════════════════════════════════════════

                     ROLLING UPDATE FLOW

Revision 1
Deployment
    │
    ▼
RS-1
    │
    ├── nginx:1.23
    ├── nginx:1.23
    └── nginx:1.23

Update Image
kubectl set image ...

    │
    ▼

Revision 2
Deployment
    │
    ├── RS-1 (Old)
    │      └── scaled down
    │
    └── RS-2 (New)
           ├── nginx:1.24
           ├── nginx:1.24
           └── nginx:1.24

═══════════════════════════════════════════════════════════════

                      DEPLOYMENT COMMANDS

Create
kubectl apply -f deploy.yaml

List Deployments
kubectl get deploy

List ReplicaSets
kubectl get rs

Scale Deployment
kubectl scale deploy nginx-deploy --replicas=5

Update Image
kubectl set image deploy/nginx-deploy \
nginx-container=nginx:1.24

Rollout Status
kubectl rollout status deploy/nginx-deploy

Rollout History
kubectl rollout history deploy/nginx-deploy

Revision Details
kubectl rollout history deploy/nginx-deploy \
--revision=2

Rollback
kubectl rollout undo deploy/nginx-deploy

Rollback to Revision
kubectl rollout undo deploy/nginx-deploy \
--to-revision=2

═══════════════════════════════════════════════════════════════

                     TROUBLESHOOTING FLOW

Deployment Not Working?
        │
        ▼

kubectl get deploy

        │
        ▼

kubectl get rs

        │
        ▼

kubectl get pods

        │
        ▼

kubectl describe deploy <name>

        │
        ▼

kubectl describe rs <name>

        │
        ▼

kubectl describe pod <name>

═══════════════════════════════════════════════════════════════

                     INTERVIEW GOLD

ReplicationController
    │
    └── Old Technology

ReplicaSet
    │
    └── Maintains Pod Count

Deployment
    │
    ├── Uses ReplicaSet Internally
    ├── Supports Rollback
    ├── Supports Rolling Updates
    └── Most Common Production Workload
```

### 🧠 30-Second Interview Summary

```text
Deployment
    ↓ manages
ReplicaSet
    ↓ manages
Pods

ReplicaSet = maintains desired pod count

Deployment = ReplicaSet + Rolling Update +
Rollback + Revision History
```

### 🔥 Commands You Must Memorize for CKA

```bash
kubectl get deploy
kubectl get rs
kubectl get pods

kubectl scale deploy nginx-deploy --replicas=5

kubectl set image deploy/nginx-deploy \
nginx-container=nginx:1.24

kubectl rollout status deploy/nginx-deploy
kubectl rollout history deploy/nginx-deploy
kubectl rollout undo deploy/nginx-deploy

kubectl describe deploy nginx-deploy
```

For **0–3 years DevOps/Kubernetes interviews**, this covers about **90% of ReplicaSet & Deployment questions**.
