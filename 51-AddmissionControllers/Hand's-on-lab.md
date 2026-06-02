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