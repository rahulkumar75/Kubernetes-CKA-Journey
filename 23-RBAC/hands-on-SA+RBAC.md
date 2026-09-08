# ServiceAccount + RBAC

A **ServiceAccount (SA)** represents an identity used by a **Pod/application**, rather than a human user.

```text
Human
  ↓
User
  ↓
RoleBinding
  ↓
Role

Pod / Application
  ↓
ServiceAccount
  ↓
RoleBinding
  ↓
Role
```

### Real-world example

Suppose a monitoring application runs in the `monitoring` namespace and needs to read Pods.

Instead of giving the application excessive permissions, create a dedicated ServiceAccount with only the required access.

---

## Step 1 — Create a ServiceAccount

```bash
kubectl create serviceaccount monitoring-sa -n monitoring
```

Verify:

```bash
kubectl get serviceaccount -n monitoring
```

---

## Step 2 — Create a Role

Create `pod-reader.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: monitoring
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

Apply:

```bash
kubectl apply -f pod-reader.yaml
```

This Role allows:

```text
get Pods
list Pods
watch Pods
```

---

## Step 3 — Bind the Role to the ServiceAccount

Create `pod-reader-binding.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: monitoring
subjects:
  - kind: ServiceAccount
    name: monitoring-sa
    namespace: monitoring
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f pod-reader-binding.yaml
```

The relationship is:

```text
ServiceAccount: monitoring-sa
          ↓
     RoleBinding
          ↓
    Role: pod-reader
          ↓
   get/list/watch Pods
```

---

## Step 4 — Verify Permissions

Check what the ServiceAccount can do:

```bash
kubectl auth can-i get pods \
  --as=system:serviceaccount:monitoring:monitoring-sa \
  -n monitoring
```

Expected:

```text
yes
```

Check a forbidden operation:

```bash
kubectl auth can-i delete pods \
  --as=system:serviceaccount:monitoring:monitoring-sa \
  -n monitoring
```

Expected:

```text
no
```

### ServiceAccount identity format

```text
system:serviceaccount:<namespace>:<serviceaccount-name>
```

Example:

```text
system:serviceaccount:monitoring:monitoring-sa
```

---

## Step 5 — Use the ServiceAccount from a Pod

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: monitoring-app
  namespace: monitoring
spec:
  serviceAccountName: monitoring-sa
  containers:
    - name: app
      image: nginx
```

Apply:

```bash
kubectl apply -f monitoring-app.yaml
```

Verify:

```bash
kubectl get pod monitoring-app -n monitoring -o jsonpath='{.spec.serviceAccountName}'
```

Output:

```text
monitoring-sa
```

The Pod now runs using the identity:

```text
monitoring-app
      ↓
monitoring-sa
      ↓
RoleBinding
      ↓
pod-reader
```

---

# Human User vs ServiceAccount

| Identity       | Used by         | Example         |
| -------------- | --------------- | --------------- |
| User           | Human           | `adam`          |
| ServiceAccount | Pod/Application | `monitoring-sa` |

Both can be granted permissions through RBAC.

```text
User ────────────────┐
                     │
ServiceAccount ──────┤
                     ↓
                RoleBinding
                     ↓
                   Role
                     ↓
                Permissions
```

### Important CKA Point

The `subjects.kind` changes:

For a human:

```yaml
subjects:
  - kind: User
    name: adam
```

For a ServiceAccount:

```yaml
subjects:
  - kind: ServiceAccount
    name: monitoring-sa
    namespace: monitoring
```

---

# CKA Memory Rule

> **User → human identity**
> **ServiceAccount → application identity**
> **Both can receive RBAC permissions through RoleBinding.**

### Most important command

```bash
kubectl auth can-i <verb> <resource> \
  --as=system:serviceaccount:<namespace>:<serviceaccount>
```

Example:

```bash
kubectl auth can-i list pods \
  --as=system:serviceaccount:monitoring:monitoring-sa \
  -n monitoring
```
