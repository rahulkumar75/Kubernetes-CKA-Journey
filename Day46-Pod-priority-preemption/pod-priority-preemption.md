# Kubernetes Pod Priority & Preemption

> **Key concept:** Evict == Preemption == Kick Out
> When a higher-priority pod needs resources, lower-priority pods are terminated (preempted) to make room.

---

## Table of Contents

1. [Why Pods Go Pending](#why-pods-go-pending)
2. [Priority Classes](#priority-classes)
3. [Key Fields](#key-fields)
4. [Implementation](#implementation)
5. [Verification & Demo](#verification--demo)
6. [Useful Commands](#useful-commands)

---

## Why Pods Go Pending

A pod enters **Pending** state when the scheduler cannot find a node with enough available resources to run it.

**Example scenario:**
- Node total capacity: 16 CPU cores, 256 GB RAM
- 3 pods already running and consuming resources
- New pod (POD4) requiring 4 CPU cores → stays **Pending** (insufficient resources)

---

## Priority Classes

Priority classes let you rank workloads so the scheduler knows which pods to protect and which to evict.

> ⚠️ Hard-coding a raw priority number directly on a pod is **not recommended**.
> Always create a named `PriorityClass` and reference it by name.

### Example tiers

| Class Name        | Value     | Use Case                        |
|-------------------|-----------|---------------------------------|
| `higher-priority` | 1,000,000 | Critical production workloads   |
| `medium-priority` | 100,000   | Standard services               |
| `low-priority`    | 10,000    | Batch jobs, dev workloads       |
| *(no class set)*  | 0         | Default if no `globalDefault` exists |

---

## Key Fields

### `value`
The integer priority number. Higher = more important.

### `globalDefault`
```yaml
globalDefault: true   # pods without a priorityClassName inherit this class
```
Only **one** PriorityClass can have `globalDefault: true`.

### `preemptionPolicy`
```yaml
preemptionPolicy: Never   # this pod will wait in Pending instead of evicting others
```
- `PreemptLowerPriority` (default) — evicts lower-priority pods to schedule this one
- `Never` — pod waits in Pending; does not preempt anyone

---

## Implementation

### Step 1 — Create Priority Classes

**`pc1.yaml` — High priority**
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: higher-priority
value: 1000000
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "Critical production workloads"
```

**`pc2.yaml` — Medium priority**
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: medium-priority
value: 100000
globalDefault: true
preemptionPolicy: PreemptLowerPriority
description: "Standard services"
```

**`pc3.yaml` — Low priority (never preempts)**
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: low-priority
value: 10000
globalDefault: false
preemptionPolicy: Never
description: "Batch and dev workloads"
```

Apply all at once:
```bash
kubectl apply -f .
```

Verify:
```bash
kubectl get pc
```

---

### Step 2 — Create Pods with a Priority Class

Add a single field to your pod spec — no need to repeat the numeric value:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-pod
spec:
  priorityClassName: higher-priority   # ← only field needed
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        cpu: "15000m"
        memory: "1Gi"
```

---

## Verification & Demo

### Create a resource-constrained environment

Deploy a dummy workload to fill the node:

```yaml
# deployment.yaml — 5 replicas, each requesting 500m CPU and 256Mi RAM
# Total consumed: ~2.5 CPU cores, ~1.5 GB RAM
```

Check node usage (requires metrics-server):
```bash
kubectl top node
kubectl describe node <node-name>
```

### Trigger preemption

Schedule a pod requesting 15 CPU cores with `higher-priority`. The scheduler will:

1. Find the node has insufficient free resources
2. Identify lower-priority pods as preemption candidates
3. Evict them to free up space
4. Schedule the critical pod
5. Move evicted pods back to **Pending**

Once the critical pod is deleted, the evicted pods reschedule automatically.

---

## Useful Commands

| Command | Purpose |
|---------|---------|
| `kubectl get pc` | List all PriorityClass objects |
| `kubectl apply -f .` | Apply all YAML files in the current directory |
| `kubectl describe node <name>` | Inspect node capacity and running pods |
| `kubectl top node` | Live CPU/memory usage (needs metrics-server) |
| `kubectl explain priorityclass` | Show API field docs inline |
| `kubectl get events --sort-by=.metadata.creationTimestamp` | View events sorted by time |
| `kubectl get events --sort-by=.metadata.creationTimestamp -w` | Watch events live |
| `kubectl describe pod <name>` | Check why a pod is Pending or was evicted |
