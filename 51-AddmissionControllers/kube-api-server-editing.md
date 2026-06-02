In a **kind cluster**, the control plane runs as a **Docker container**, not as a systemd service like on a kubeadm cluster.

### 1. Find the control-plane container

```bash
docker ps
```

Example output:

```bash
CONTAINER ID   IMAGE                  NAMES
abc123         kindest/node:v1.32.0   kind-control-plane
```

---

### 2. Exec into the control-plane node

```bash
docker exec -it kind-control-plane bash
```

If `bash` doesn't exist:

```bash
docker exec -it kind-control-plane sh
```

---

### 3. Locate kube-apiserver manifest

Inside the container:

```bash
cd /etc/kubernetes/manifests
ls
```

Output:

```bash
etcd.yaml
kube-apiserver.yaml
kube-controller-manager.yaml
kube-scheduler.yaml
```

View:

```bash
cat kube-apiserver.yaml
```

Edit:

```bash
vi kube-apiserver.yaml
```

or

```bash
sed ...
```

---

### 4. Verify static pod restart

The kubelet inside the Kind node watches:

```bash
/etc/kubernetes/manifests
```

After saving the file:

```bash
kubectl get pods -n kube-system -w
```

You'll see:

```bash
kube-apiserver-kind-control-plane
```

restart automatically.

---

### 5. Check the new arguments

```bash
kubectl get pod -n kube-system kube-apiserver-kind-control-plane -o yaml
```

or

```bash
kubectl describe pod -n kube-system kube-apiserver-kind-control-plane
```

---

### Example: Enable Audit Logging

Edit:

```yaml
- --audit-log-path=/var/log/kubernetes/audit.log
- --audit-policy-file=/etc/kubernetes/audit-policy.yaml
```

Save the file.

Then verify:

```bash
kubectl logs -n kube-system kube-apiserver-kind-control-plane
```

---

### Faster way (one-liner)

```bash
docker exec -it $(docker ps --filter "name=control-plane" --format "{{.Names}}") bash
```

---

### For Admission Controller labs

To see currently enabled admission plugins:

```bash
kubectl -n kube-system get pod kube-apiserver-kind-control-plane \
-o yaml | grep enable-admission-plugins
```

Or directly:

```bash
docker exec -it kind-control-plane \
grep enable-admission-plugins \
/etc/kubernetes/manifests/kube-apiserver.yaml
```

This is typically what you'll modify when practicing:

* `ValidatingAdmissionPolicy`
* `OPA Gatekeeper`
* `Kyverno`
* `Pod Security Admission`
* Custom Admission Webhooks
* ImagePolicyWebhook

on a Kind cluster.
