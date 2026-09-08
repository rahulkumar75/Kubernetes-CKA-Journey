
# Hands-On Practice

## Step 1 — Create ConfigMap

```bash
kubectl create cm app-cm \
  --from-literal=firstname=rahul \
  --from-literal=lastname=kumar
```

Verify:

```bash
kubectl get cm app-cm
kubectl describe cm app-cm
```

---

## Step 2 — Inject ConfigMap Using `envFrom`

Create:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-env
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      envFrom:
        - configMapRef:
            name: app-cm
```

Apply:

```bash
kubectl apply -f cm-env.yaml
```

Verify:

```bash
kubectl exec cm-env -- env
```

---

## Step 3 — Inject One Key

```yaml
env:
  - name: FIRST_NAME
    valueFrom:
      configMapKeyRef:
        name: app-cm
        key: firstname
```

Verify:

```bash
kubectl exec <pod-name> -- sh -c 'echo $FIRST_NAME'
```

---

## Step 4 — Mount ConfigMap as Files

```yaml
volumes:
  - name: config
    configMap:
      name: app-cm
```

```yaml
volumeMounts:
  - name: config
    mountPath: /etc/config
    readOnly: true
```

Verify:

```bash
kubectl exec <pod-name> -- ls /etc/config
kubectl exec <pod-name> -- cat /etc/config/firstname
```

---

## Step 5 — Create Secret

```bash
kubectl create secret generic app-secret \
  --from-literal=username=admin \
  --from-literal=password='mypassword'
```

Verify:

```bash
kubectl get secret
kubectl describe secret app-secret
```

---

## Step 6 — Inject Secret

```yaml
env:
  - name: APP_USERNAME
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: username
```

Verify:

```bash
kubectl exec <pod-name> -- sh -c 'echo $APP_USERNAME'
```

---

# 15. CKA Troubleshooting

### Pod shows `CreateContainerConfigError`

Check:

```bash
kubectl describe pod <pod-name>
```

Common causes:

* ConfigMap does not exist
* Secret does not exist
* Key name is incorrect
* Wrong namespace

---

### Environment variable has old value

Remember:

```text
ConfigMap/Secret → env → existing container does NOT auto-update
```

Restart the Pod/Deployment.

---

### Mounted ConfigMap file has old value

Check:

```bash
kubectl exec <pod-name> -- cat /etc/config/<key>
```

Remember:

* Normal ConfigMap volume → eventually updates
* `subPath` mount → does not automatically update

---



