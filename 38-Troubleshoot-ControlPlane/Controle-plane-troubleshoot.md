# Day 38 — Troubleshooting Control Plane Failures in Kubernetes

> **Key tool:** `crictl` — a client for containerd (used instead of Docker in k8s > v1.24).
> It is pre-installed in the CKA exam environment.

```bash
crictl ps        # list running containers
crictl ps -a     # include exited containers
```

---

## Scenario 1 — API Server Not Running (Bad Manifest Command)

### Symptoms
- `crictl ps` shows the API server is not running.

### Diagnosis

**Step 1 — Check logs**
```bash
crictl ps -a | grep api          # find the exited API server container
crictl logs <container-id>       # inspect its logs
```

If nothing useful is found in `crictl logs`, check the default log directory:
```bash
ls -lrt /var/log/containers/ | grep "kube-apiserver"
```

**Step 2 — Inspect the manifest**
```bash
vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Look at the `command` field — verify the binary name is exactly `kube-apiserver`. A typo here (e.g., `kube-apiserver-typo`) will prevent the API server from starting.

### Fix
Correct the command value in the manifest. The kubelet will automatically restart the static pod.

### Verify
```bash
kubectl get nodes
```

---

## Scenario 2 — API Server Running but `kubectl` Still Fails

### Symptoms
- API server container is running.
- `kubectl` commands still return errors.

### Root Cause
Wrong or missing kubeconfig — `kubectl` cannot authenticate with the API server.

### Diagnosis
```bash
# Check default kubeconfig location
ls -ltr $HOME/.kube/

# Check the admin config
ls -ltr /etc/kubernetes/
```

### Fix

**Temporary** — pass kubeconfig explicitly:
```bash
kubectl get nodes --kubeconfig /etc/kubernetes/admin.conf
```

**Permanent** — copy `admin.conf` into the default kubeconfig location:
```bash
cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

---

## Scenario 3 — Pod Stuck in Pending (Scheduler Down)

### Symptoms
- Pod created successfully but stays in `Pending` state.
- `kubectl describe pod <name>` shows no events and no node assignment.

### Root Cause
The **kube-scheduler** is not running — it is responsible for assigning pods to nodes.

### Diagnosis
```bash
kubectl get pods -n kube-system          # check all control plane pods
kubectl logs kube-scheduler-master -n kube-system
kubectl describe pod kube-scheduler-master -n kube-system
```

Common finding: `ErrImagePull` — the scheduler pod is using an incorrect image tag.

### Fix
```bash
sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml
```

Correct the image name/tag, save the file, and the kubelet will restart the pod automatically.

### Verify
```bash
kubectl get pods -n kube-system          # scheduler should be Running
kubectl get pod <your-pod>               # should move from Pending → Running
```

---

## Scenario 4 — Deleted Pod Not Recreated (Controller Manager Down)

### Symptoms
- A deployment has 2 replicas, but after deleting one pod, only 1 pod remains.
- The deployment is not self-healing.

### Root Cause
The **kube-controller-manager** is down. It is responsible for:
- Maintaining the desired replica count
- Reconciling actual state vs. desired state
- Handling scaling operations

### Diagnosis
```bash
kubectl get pods -n kube-system          # controller-manager likely in CrashLoopBackOff
kubectl logs kube-controller-manager -n kube-system
kubectl describe pod kube-controller-manager -n kube-system
```

Common findings:
- **Executable not found** — typo in the `command` field of the manifest.
- **TLS certificate errors** — volume mounts for certs are misconfigured.

### Fix A — Bad Command
```bash
vi /etc/kubernetes/manifests/kube-controller-manager.yaml
```

Verify the `command` field starts with `kube-controller-manager` (no typos).

### Fix B — Cert Volume Mount Mismatch
In the same manifest file, verify the `volumeMounts` and `volumes` sections:

```yaml
volumeMounts:
  - mountPath: /etc/kubernetes/pki
    name: k8s-certs          # ← must match the volume name below

volumes:
  - name: k8s-certs          # ← must match the volumeMount name above
    hostPath:
      path: /etc/kubernetes/pki
```

> **Rule:** The `name` in `volumeMounts` must exactly match the `name` in `volumes`.

### Verify
```bash
kubectl get pods -n kube-system          # controller-manager should be Running
kubectl get pods                         # deployment should restore to desired replica count
```

---

## Scenario 5 — Scaling Fails (Controller Manager Cert Issue)

### Symptoms
- `kubectl scale` command appears to succeed but the replica count does not change.

### Root Cause
Same component as Scenario 4 — the kube-controller-manager is unable to start due to incorrect certificate volume paths. See **Fix B** above.

---

## Quick Reference — Control Plane Health Check

```bash
# Cluster-level status
kubectl cluster-info

# All control plane pod statuses
kubectl get pods -n kube-system

# Individual component logs
crictl logs <container-id>
kubectl logs <pod-name> -n kube-system

# Manifest files (static pods)
ls /etc/kubernetes/manifests/
```

| Component | Responsible For | Manifest Location |
|---|---|---|
| `kube-apiserver` | All API interactions | `/etc/kubernetes/manifests/kube-apiserver.yaml` |
| `kube-scheduler` | Assigning pods to nodes | `/etc/kubernetes/manifests/kube-scheduler.yaml` |
| `kube-controller-manager` | Replica count, self-healing, scaling | `/etc/kubernetes/manifests/kube-controller-manager.yaml` |

---

## General Troubleshooting Flow

```
1. crictl ps -a | grep <component>     → Is the container running or exited?
2. crictl logs <container-id>          → What error is it throwing?
3. kubectl logs <pod> -n kube-system   → Same, via kubectl if API is available
4. kubectl describe pod <pod> -n kube-system  → Check Events section
5. vi /etc/kubernetes/manifests/<component>.yaml  → Fix the root cause
```

> **Exam tip:** `crictl` is essential when the API server itself is down and `kubectl` is unavailable.