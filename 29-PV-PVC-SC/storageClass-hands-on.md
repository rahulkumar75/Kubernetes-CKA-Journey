**Dynamic Provisioning in KIND using local-path provisioner**
This is modern Kubernetes style.

PVC automatically creates PV.

---

# Why Needed?

Instead of manually creating PV every time:

```text id="jlwmt8"
PVC → StorageClass → Auto PV
```

---

# Step 1️⃣ Install local-path provisioner

```bash id="jlwmt9"
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/master/deploy/local-path-storage.yaml
```

Check:

```bash id="jlwmu1"
kubectl get pods -n local-path-storage
```

Should show Running.

---

# Step 2️⃣ Check StorageClass

```bash id="jlwmu2"
kubectl get sc
```

Expected:

```text id="jlwmu3"
local-path
```

---

# Step 3️⃣ Create PVC Only

```yaml id="jlwmu4"
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dynamic-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-path
  resources:
    requests:
      storage: 1Gi
```

```bash id="jlwmu5"
kubectl apply -f pvc.yaml
```

---

# Step 4️⃣ Watch Auto PV Creation

```bash id="jlwmu6"
kubectl get pv,pvc
```

You’ll see:

```text id="jlwmu7"
PV auto-created
PVC Bound
```

---

# Step 5️⃣ Use in Pod

```yaml id="jlwmu8"
apiVersion: v1
kind: Pod
metadata:
  name: app-dynamic
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - mountPath: /data
      name: store
  volumes:
  - name: store
    persistentVolumeClaim:
      claimName: dynamic-pvc
```

---

# Step 6️⃣ Verify

```bash id="jlwmu9"
kubectl exec -it app-dynamic -- sh
echo dynamic > /data/test.txt
cat /data/test.txt
```

---

# 🔥 How It Works Internally

```text id="jlwmv1"
PVC created
↓
StorageClass local-path called
↓
Provisioner creates hostPath/local volume
↓
PV auto-bound
↓
Pod mounts PVC
```

---

# 🚨 Troubleshooting Dynamic Provisioning

## PVC Pending

Check:

```bash id="jlwmv2"
kubectl describe pvc dynamic-pvc
```

Possible causes:

* provisioner pod not running
* wrong storageClass name
* no default SC

---

# CKA Interview Answer

**Static provisioning:** Admin creates PV manually.
**Dynamic provisioning:** StorageClass automatically provisions PV when PVC is created.

---

# ⚡ CKA Memory Trick

```text id="jlwmv3"
PV = Manual
PVC = Request
SC = Automatic Factory
```

---

# 🔥 Real World

In EKS:

* gp3 StorageClass → EBS volume auto-created

In AKS:

* managed-csi

In GKE:

* standard-rwo

---

