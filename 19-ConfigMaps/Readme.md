# 19 - Kubernetes ConfigMap and Secret

## 🎯 Objective

Understand how to:

* Store non-sensitive configuration using **ConfigMaps**
* Store sensitive data using **Secrets**
* Inject configuration through **environment variables** and **volumes**
* Use `envFrom`, `configMapKeyRef`, and `secretKeyRef`
* Understand **ConfigMap/Secret update behavior**
* Create and troubleshoot ConfigMaps and Secrets using imperative and declarative methods

---

# 1. ConfigMap

A **ConfigMap** stores non-sensitive configuration data as **key-value pairs**.

Instead of hardcoding configuration inside a Pod manifest:

```text
Pod
 ├── APP_ENV=prod
 ├── APP_PORT=8080
 └── LOG_LEVEL=info
```

Store it separately:

```text
ConfigMap
 ├── APP_ENV=prod
 ├── APP_PORT=8080
 └── LOG_LEVEL=info
        ↓
       Pod
```

### Why use ConfigMap?

* Keeps configuration separate from application/container definitions
* Allows the same configuration to be reused by multiple Pods
* Makes configuration easier to update and manage

> **Important:** ConfigMaps are for **non-sensitive** data. Do not store passwords, tokens, or credentials in them.

---

# 2. ConfigMap — Important Rules

* ConfigMap and Pod must be in the **same namespace**.
* A ConfigMap can be consumed by multiple Pods.
* ConfigMap data can be injected as:

  * Individual environment variables
  * All environment variables
  * Files through a volume
* A **static Pod** cannot directly reference ConfigMaps or other API objects.

---

# 3. Create ConfigMap — Imperative

### Using literals

```bash
kubectl create cm app-cm \
  --from-literal=firstname=rahul \
  --from-literal=lastname=kumar
```

Verify:

```bash
kubectl get cm
kubectl describe cm app-cm
```

View the YAML:

```bash
kubectl get cm app-cm -o yaml
```

### From a file

```bash
kubectl create cm app-cm --from-file=app.config
```

The filename becomes the ConfigMap key.

---

# 4. Inject ConfigMap as Environment Variables

There are two common approaches.

## A. `envFrom` → Import everything

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      envFrom:
        - configMapRef:
            name: app-cm
```

Every key in `app-cm` becomes an environment variable.

Verify:

```bash
kubectl exec configmap-env -- env
```

Or:

```bash
kubectl exec configmap-env -- sh -c 'echo $firstname'
```

### Memory rule

```text
envFrom → import ALL keys
```

---

# 5. `configMapKeyRef` → Import One Key

Use this when the Pod needs only a specific ConfigMap key.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-key
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      env:
        - name: FIRSTNAME
          valueFrom:
            configMapKeyRef:
              name: app-cm
              key: firstname
```

Here:

```text
ConfigMap key      → firstname
Container variable → FIRSTNAME
Value              → rahul
```

Verify:

```bash
kubectl exec configmap-key -- sh -c 'echo $FIRSTNAME'
```

### Memory rule

```text
configMapKeyRef → import ONE key
```

---

# 6. ConfigMap as a Volume

A ConfigMap can also be mounted as files.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-volume
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: config
          mountPath: /etc/config
          readOnly: true

  volumes:
    - name: config
      configMap:
        name: app-cm
```

Each ConfigMap key becomes a file:

```text
/etc/config/
├── firstname
└── lastname
```

Verify:

```bash
kubectl exec configmap-volume -- ls /etc/config
```

Read a value:

```bash
kubectl exec configmap-volume -- cat /etc/config/firstname
```

### Memory rule

```text
ConfigMap Volume → keys become files
```

---

# 7. ConfigMap Update Behavior

This is an important CKA concept.

### Environment variables

If a ConfigMap is injected using:

```yaml
envFrom:
```

or:

```yaml
configMapKeyRef:
```

the existing container's environment variables **do not automatically update** when the ConfigMap changes.

You normally need to **restart/recreate the Pod**.

For a Deployment:

```bash
kubectl rollout restart deployment <deployment-name>
```

---

### Volume-mounted ConfigMap

When mounted as a normal volume, Kubernetes periodically updates the mounted files after the ConfigMap changes.

Example:

```bash
kubectl edit cm app-cm
```

Then check:

```bash
kubectl exec configmap-volume -- cat /etc/config/firstname
```

> The update is **eventually consistent**, not necessarily instantaneous.

> **Important:** ConfigMap volumes mounted using `subPath` do **not** receive automatic updates.

---

# 8. Declarative ConfigMap

Generate YAML from an imperative command:

```bash
kubectl create cm app-cm \
  --from-literal=firstname=rahul \
  --from-literal=lastname=kumar \
  --dry-run=client -o yaml > cm.yaml
```

Then:

```bash
kubectl apply -f cm.yaml
```

Verify:

```bash
kubectl get cm app-cm -o yaml
```

---

# 9. Immutable ConfigMap

A ConfigMap can be made immutable:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-cm
data:
  APP_ENV: production

immutable: true
```

Once:

```yaml
immutable: true
```

the `data` and `binaryData` fields cannot be modified.

To change it:

```bash
kubectl delete cm app-cm
kubectl apply -f cm.yaml
```

> **CKA:** Immutable ConfigMap → **delete and recreate** to change its contents.

---

# 10. Secret

A **Secret** stores sensitive configuration such as:

* Passwords
* API tokens
* Usernames
* Certificates
* Credentials

Example:

```text
Secret
 ├── username
 └── password
       ↓
      Pod
```

> **Important:** Secret data is commonly represented as **base64**, but base64 is **encoding, not encryption**.

Do not treat a Kubernetes Secret as automatically secure simply because it is a Secret.

---

# 11. Create Secret — Imperative

```bash
kubectl create secret generic backend-user \
  --from-literal=backend-username=backend-admin
```

Verify:

```bash
kubectl get secret
kubectl describe secret backend-user
```

View the Secret:

```bash
kubectl get secret backend-user -o yaml
```

The value appears encoded:

```yaml
data:
  backend-username: <base64-value>
```

Decode when required:

```bash
echo '<base64-value>' | base64 --decode
```

---

# 12. Inject Secret into a Pod

Use `secretKeyRef` to inject a single Secret key.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: backend-user
              key: backend-username
```

Apply:

```bash
kubectl apply -f secret-pod.yaml
```

Verify:

```bash
kubectl exec secret-demo -- sh -c 'echo $SECRET_USERNAME'
```

Output:

```text
backend-admin
```

### Memory rule

```text
secretKeyRef → import ONE Secret key
```

---

# 13. Secret vs ConfigMap

| Feature                    | ConfigMap                   | Secret                  |
| -------------------------- | --------------------------- | ----------------------- |
| Purpose                    | Non-sensitive configuration | Sensitive configuration |
| Example                    | APP_ENV, LOG_LEVEL          | Password, token         |
| Environment injection      | ✅                           | ✅                       |
| Volume mount               | ✅                           | ✅                       |
| Base64 required            | ❌                           | Commonly used           |
| Encryption by base64       | ❌                           | ❌                       |
| Same namespace requirement | ✅                           | ✅                       |



### Easy Memory Rule

```text
ConfigMap → Configuration
Secret    → Sensitive data
```

| Method | Auto Update? | Restart Needed? |
| --- | --- | --- |
| Volume Mount | ✅ YES | ❌ No |
| Environment Variable | ❌ NO | ✅ Yes |

---
