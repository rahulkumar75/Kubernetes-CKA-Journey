# Kubernetes Admission Controllers


## 1. What Is an Admission Controller?

An Admission Controller is a **built-in plugin** — a piece of code — that **intercepts requests to the Kubernetes API server** after authentication and authorization have succeeded, but before the object is persisted to etcd.

| Property | Details |
|---|---|
| Triggers on | CREATE, DELETE, or MODIFY operations only |
| Position in pipeline | After AuthN + AuthZ; before etcd write |
| Plugin type | Built-in (compiled into kube-apiserver) |
| Custom extension | Admission Webhooks (external HTTP servers) |

---

## 2. The Three Admission Controller Types

### 2.1 Mutating Admission Webhook

- Runs in **Phase 1**.
- Can **modify (mutate)** the incoming object before it is validated.
- Example use-cases: injecting sidecar containers, setting default resource limits, adding labels.
- Runs before validating controllers so validation sees the final, mutated object.

### 2.2 Validating Admission Policy (CEL-based)

- Runs in **Phase 2a**.
- Uses **Common Expression Language (CEL)** rules embedded directly into the cluster — no external webhook required.
- **Read-only**: can only approve or reject; cannot modify the object.
- Introduced as a stable feature in Kubernetes v1.30.

### 2.3 Validating Admission Webhook

- Runs in **Phase 2b**.
- Calls an external HTTPS webhook server to approve or reject the request.
- **Cannot modify** the object — validation only.
- Example: reject pods in namespaces that do not exist.

### Summary Table

| Type | Action | Phase | Example |
|---|---|---|---|
| Mutating Webhook | Modifies the object | Phase 1 | LimitRanger (injects defaults) |
| Validating Policy | Validates via CEL rules | Phase 2a | Deny pods without labels |
| Validating Webhook | Validates only; no changes | Phase 2b | NamespaceLifecycle |

---

## 3. Request Flow — End to End

Every API request travels through the following pipeline. Both phases must succeed before the object is written to etcd.

```
┌─────────────────────────────────────────┐
│         kubectl / API Client            │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│    API Server — Auth (AuthN + AuthZ)    │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│  Phase 1 — Mutating Admission Webhooks  │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│        Object Schema Validation         │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│  Phase 2a — Validating Admission        │
│             Policies (CEL)              │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│  Phase 2b — Validating Admission        │
│             Webhooks                    │
└────────────────────┬────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────┐
│       Persisted to etcd  ✓              │
└─────────────────────────────────────────┘
```

> ❌ **If Phase 1 is rejected** — the entire request is rejected immediately. Phase 2 is never reached.

> ❌ **If Phase 2a or Phase 2b is rejected** — either one failing rejects the entire request.

---

## 4. Admission Webhooks In Depth

Webhooks allow Kubernetes admission controllers to call external services. There are two ways to reference a webhook endpoint:

### 4.1 Webhook Contact Methods

| Method | Description |
|---|---|
| **URL (DNS)** | Provide an HTTPS URL pointing to the webhook server. Suitable when the webhook runs outside the cluster. |
| **Service Reference** | Reference an in-cluster Service (ClusterIP). The webhook pod is exposed via a Kubernetes Service whose endpoint is used as the target. |

> ⚠️ **Warning:** If webhook pods are down or the Service has no healthy endpoints, requests will fail with `Connection refused` or `Timeout`. Always ensure your webhook deployment has liveness/readiness probes and is highly available.

### 4.2 Failure Policy

Each webhook configuration specifies a `failurePolicy`:

- **`Fail`** — If the webhook cannot be reached, the API request is **denied**. (Safer, recommended for production.)
- **`Ignore`** — If the webhook cannot be reached, the request **proceeds**. (Useful during initial rollout.)

---

## 5. Why Use Admission Controllers?

- Enforce security policies (e.g., ban privileged containers cluster-wide).
- Auto-inject sidecar proxies (Istio, Linkerd) without changing app manifests.
- Set default resource requests/limits so pods are always scheduled correctly.
- Prevent resource quota violations before they happen.
- Require specific labels on every workload for governance/chargeback.
- Restrict container image registries to approved internal registries only.

---

## 6. Default Built-In Admission Controllers

The following controllers are enabled by default in a standard kubeadm cluster.

| Controller | Type | Description |
|---|---|---|
| `NamespaceLifecycle` | Validating | Rejects resources in non-existent or terminating namespaces |
| `NamespaceAutoProvision` | Mutating | Auto-creates a namespace if it does not exist (**not default**) |
| `LimitRanger` | Mutating | Injects default CPU/memory requests & limits into pods |
| `ResourceQuota` | Validating | Enforces hard resource quotas per namespace |
| `ServiceAccount` | Mutating | Auto-assigns a ServiceAccount to pods |
| `DefaultStorageClass` | Mutating | Assigns a default StorageClass to PVCs that omit one |
| `DefaultTolerationSeconds` | Mutating | Adds default tolerations for node taints |
| `NodeRestriction` | Validating | Limits what kubelet can label or modify |

To see which admission controllers are currently enabled:

```bash
kubectl get pod kube-apiserver-controlplane -n kube-system -o yaml | grep -i admission
```

---

## 7. Hands-On Lab

### 7.1 Enable NamespaceAutoProvision

By default, `NamespaceLifecycle` blocks pod creation in non-existent namespaces. This exercise adds `NamespaceAutoProvision` so the namespace is auto-created.

**Step 1 — Observe the default rejection**

```yaml
# pod-in-missing-namespace.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
  namespace: test   # does not exist
spec:
  containers:
  - name: nginx
    image: nginx:1.20
```

```bash
kubectl apply -f pod-in-missing-namespace.yaml
# Error from server (NotFound): namespaces "test" not found
```

**Step 2 — Back up the API server manifest**

```bash
# SSH to your control-plane node first
sudo cp /etc/kubernetes/manifests/kube-apiserver.yaml \
        /etc/kubernetes/manifests/kube-apiserver.yaml.backup
```

**Step 3 — Edit the manifest**

```bash
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml

# Find the flag and append NamespaceAutoProvision:
# Before:
- --enable-admission-plugins=NodeRestriction,ResourceQuota,LimitRanger

# After:
- --enable-admission-plugins=NodeRestriction,ResourceQuota,LimitRanger,NamespaceAutoProvision
```

**Step 4 — Watch the API server restart**

```bash
kubectl -n kube-system get pods -w | grep kube-apiserver
```

**Step 5 — Verify and re-test**

```bash
kubectl apply -f pod-in-missing-namespace.yaml
kubectl get namespaces        # 'test' namespace now exists
kubectl get pods -n test      # pod is running
```

> 💡 **Tip:** If the API server does not restart automatically, move the manifest to a temp location and back to `/etc/kubernetes/manifests/` to force a restart.

---

### 7.2 ResourceQuota — Validating Controller Demo

**Step 1 — Create namespace and quota**

```yaml
# resource-quota-demo.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: quota-demo
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: quota-demo
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
    pods: "4"
```

```bash
kubectl apply -f resource-quota-demo.yaml
```

**Step 2 — Attempt to exceed the quota**

```yaml
# big-pod.yaml
spec:
  containers:
  - name: big-container
    image: nginx
    resources:
      requests:
        memory: "2Gi"   # exceeds namespace quota
        cpu: "1"
```

```bash
kubectl apply -f big-pod.yaml
```

> ❌ The `ResourceQuota` admission controller will reject this pod because the memory request (2Gi) exceeds the namespace limit (1Gi).

---

### 7.3 LimitRanger — Mutating Controller Demo

**Step 1 — Create a LimitRange**

```yaml
# limit-range-demo.yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-limit-range
  namespace: quota-demo
spec:
  limits:
  - default:
      memory: "512Mi"
      cpu: "200m"
    defaultRequest:
      memory: "256Mi"
      cpu: "100m"
    type: Container
```

```bash
kubectl apply -f limit-range-demo.yaml
```

**Step 2 — Create a pod with no resource spec**

```yaml
# simple-pod.yaml — no resources block at all
apiVersion: v1
kind: Pod
metadata:
  name: simple-pod
  namespace: quota-demo
spec:
  containers:
  - name: simple-container
    image: nginx
```

```bash
kubectl apply -f simple-pod.yaml
kubectl get pod simple-pod -n quota-demo -o yaml | grep -A 10 resources
```

> ✅ `LimitRanger` automatically injects `requests.memory=256Mi`, `requests.cpu=100m`, `limits.memory=512Mi`, `limits.cpu=200m` — even though they were not specified in the manifest. This is **mutating admission** in action.

---

## 8. Quick Reference Cheatsheet

| Command | Purpose |
|---|---|
| `kubectl get pod kube-apiserver-<node> -n kube-system -o yaml \| grep admission` | View enabled admission plugins |
| `ps aux \| grep kube-apiserver \| grep enable-admission` | Check plugins via process list (on node) |
| `kubectl get validatingwebhookconfigurations` | List validating webhooks in cluster |
| `kubectl get mutatingwebhookconfigurations` | List mutating webhooks in cluster |
| `kubectl describe resourcequota -n <ns>` | Inspect quota usage in a namespace |
| `kubectl describe limitrange -n <ns>` | View LimitRange defaults in a namespace |

---

## 9. CKA Exam Tips

> 📝 **Things to remember for the exam:**

- Admission controllers run **after AuthN/AuthZ** — they are the last gate before etcd.
- **Mutating always runs before Validating** so the validator sees the final object.
- `NamespaceAutoProvision` is **NOT enabled by default** — you must add it to `--enable-admission-plugins`.
- `LimitRanger` is **Mutating**. `ResourceQuota` is **Validating**.
- If a webhook server is unreachable: `failurePolicy: Fail` blocks the request; `Ignore` allows it.
- Both Phase 2a and 2b are independent — failing **either one** rejects the request.
- Always **back up** `kube-apiserver.yaml` before editing; a syntax error will crash the API server.

---

* End — Kubernetes Admission Controllers