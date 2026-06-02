# 🧠 Kubernetes Init Containers — Mind Map

## 🚀 One-Line Interview Answer

**"Init Containers are used to perform initialization tasks before the main application starts. They run sequentially, must complete successfully, and are commonly used for dependency checks, configuration setup, and database migrations."**

```text
                               ┌──────────────────────┐
                               │   INIT CONTAINERS    │
                               └──────────┬───────────┘
                                          │
          ┌───────────────────────────────┼───────────────────────────────┐
          │                               │                               │
          ▼                               ▼                               ▼

 ┌─────────────────┐            ┌─────────────────┐             ┌─────────────────┐
 │   WHAT IS IT?   │            │   USE CASES     │             │   LIFECYCLE     │
 └────────┬────────┘            └────────┬────────┘             └────────┬────────┘
          │                              │                               │
          │                              │                               │
          ▼                              ▼                               ▼

 • Special container           • Wait for DB/Service          1. Pod Created
 • Runs before app             • Download configs             2. Init Container Starts
 • Setup/Initialization        • Check dependencies          3. Task Completes
 • Must complete               • Run migrations              4. Next Init Container
 • Executes once               • Create files/directories    5. Main Container Starts


                                          │
                                          ▼

                         ┌─────────────────────────────┐
                         │   EXECUTION BEHAVIOR        │
                         └─────────────┬───────────────┘
                                       │
                                       ▼

                         • Runs sequentially
                         • One by one
                         • Next starts only after previous succeeds
                         • Main container waits
                         • Failure = Pod not started

Example:

Init-1 → Success
      ↓
Init-2 → Success
      ↓
Main App → Running



          ┌───────────────────────────────┼───────────────────────────────┐
          │                               │                               │
          ▼                               ▼                               ▼

 ┌─────────────────┐            ┌─────────────────┐             ┌─────────────────┐
 │ INTERVIEW Q&A   │            │ TROUBLESHOOTING │             │ YAML LOCATION   │
 └────────┬────────┘            └────────┬────────┘             └────────┬────────┘
          │                              │                               │
          ▼                              ▼                               ▼

 Q. Difference between          Pod Stuck in Init:0/1?       spec:
 Init & Sidecar?                ├─ Service not found           initContainers:
                                ├─ DNS issue                   - name: init
 Init → Runs once               ├─ Wrong command                ...
 Sidecar → Runs forever         ├─ Permission issue
                                └─ Infinite loop              containers:
 Q. Can multiple init                                            - name: app
 containers exist?
 Yes, sequentially.

 Q. What if init fails?
 Pod remains Pending.



                                          │
                                          ▼

                         ┌─────────────────────────────┐
                         │     USEFUL COMMANDS         │
                         └─────────────┬───────────────┘
                                       │
                                       ▼

 kubectl get pods

 kubectl describe pod <pod-name>

 kubectl logs <pod-name> -c <init-container>

 kubectl get events

 kubectl get pod <pod-name> -o yaml



                                          │
                                          ▼

                         ┌─────────────────────────────┐
                         │      COMMON EXAMPLES        │
                         └─────────────┬───────────────┘
                                       │
                                       ▼

 Wait for Service

 until nslookup myservice.default.svc.cluster.local
 do
   echo waiting...
   sleep 2
 done


 Download Config

 wget http://config-server/config.yaml


 Database Migration

 python migrate.py


 Create Required Directory

 mkdir -p /app/data
```

---

# 🎯 Interview Quick Revision (30 Seconds)

### What is an Init Container?

A special container that runs **before the application container** to perform setup tasks. It must complete successfully before the main container starts.

### Why use it?

* Wait for dependencies
* Run migrations
* Download configuration
* Prepare environment

### Key Points

✅ Runs before app container
✅ Runs only once
✅ Executes sequentially
✅ Must succeed to start Pod
✅ Failure keeps Pod in `Init:x/y`

### Most Common Debug Command

```bash
kubectl logs <pod-name> -c <init-container-name>
```

### Most Common Interview Scenario

```text
Pod Status = Init:0/1

Reason:
✓ Service not found
✓ DNS issue
✓ Wrong command
✓ Infinite loop

Fix:
kubectl describe pod
kubectl logs -c <init-container>
```

---

