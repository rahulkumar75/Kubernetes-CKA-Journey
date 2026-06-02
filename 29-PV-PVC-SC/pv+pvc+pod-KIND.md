Since you're using **kind cluster on MacBook**, normal Mac path like:

```text
/Users/rahulkumar/...
```

won’t work directly because **Kubernetes node = Docker container**, not your Mac host.

So here is a **100% working PV + PVC + Pod setup for KIND on Mac**.

---

# 🧠 Architecture

```text
Mac Host
   ↓
Docker Container (kind-control-plane) = Kubernetes Node
   ↓
hostPath PV
   ↓
PVC
   ↓
Pod
```

---

# Step 1️⃣ Create Directory Inside KIND Node

Check node container name:

```bash id="n8v2ut"
docker ps
```

Usually:

```text
kind-control-plane
```

Create folder inside node:

```bash id="jlr1pd"
docker exec -it kind-control-plane mkdir -p /tmp/pv-data
```

---

# Step 2️⃣ Create PV

```yaml id="ulhjqm"
apiVersion: v1
kind: PersistentVolume
metadata:
  name: task-pv-volume

spec:
  capacity:
    storage: 1Gi

  accessModes:
  - ReadWriteOnce

  persistentVolumeReclaimPolicy: Retain

  storageClassName: manual

  hostPath:
    path: /tmp/pv-data
```

Apply:

```bash id="pbjkrm"
kubectl apply -f pv.yaml
```

---

# Step 3️⃣ Create PVC

```yaml id="p2l6lf"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: task-pv-claim

spec:
  accessModes:
  - ReadWriteOnce

  resources:
    requests:
      storage: 500Mi

  storageClassName: manual
```

Apply:

```bash id="u2q2w6"
kubectl apply -f pvc.yaml
```

Check:

```bash id="rmy8j7"
kubectl get pv,pvc
```

Should show:

```text
PV   Bound
PVC  Bound
```

---

# Step 4️⃣ Create Pod Using PVC

```yaml id="ut9vrv"
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pv-pod

spec:
  containers:
  - name: nginx
    image: nginx
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: my-storage

  volumes:
  - name: my-storage
    persistentVolumeClaim:
      claimName: task-pv-claim
```

Apply:

```bash id="l3ky72"
kubectl apply -f pod.yaml
```

---

# Step 5️⃣ Test Persistence

Enter pod:

```bash id="8mkcvl"
kubectl exec -it nginx-pv-pod -- sh
```

Create file:

```bash id="6hwp32"
echo hello > /usr/share/nginx/html/index.html
exit
```

Delete pod:

```bash id="e6q5lw"
kubectl delete pod nginx-pv-pod
kubectl apply -f pod.yaml
```

Now check again:

```bash id="08p8nl"
kubectl exec -it nginx-pv-pod -- cat /usr/share/nginx/html/index.html
```

Output:

```text
hello
```

✅ Data persisted.

---

# 🔥 Why Mac Path Doesn't Work Directly

Because:

```text
kubectl talks to kind node container
```

Node container cannot see your Mac `/Users/...` path unless mounted.

---

# 🧪 CKA Exam Tip

In exam use:

* hostPath for quick local lab
* know PVC binding
* know pod mount syntax

---

# ⚡ Important Commands

```bash id="46c8jd"
kubectl get pv
kubectl get pvc
kubectl describe pvc task-pv-claim
kubectl get pod
```

---

# 🧠 Interview Answer

In KIND, `hostPath` refers to node container filesystem, not laptop filesystem.

---

