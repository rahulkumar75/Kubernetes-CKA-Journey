# Enterprise Kubernetes NetworkPolicy Lab 2

## Multi-Namespace Zero Trust Setup (Dev / Prod / Monitoring)

This is a **real company style lab** where we isolate environments and only allow required traffic.

---

# Architecture

```text
Internet
   ↓
Ingress
   ↓
Frontend (prod)
   ↓
Backend (prod)
   ↓
MySQL (prod)

Monitoring namespace:
Prometheus scrapes metrics

Dev namespace:
Used for testing, should NOT access prod DB
```

---

# Security Goals

| Source                | Destination          | Allow |
| --------------------- | -------------------- | ----- |
| prod frontend         | prod backend         | ✅     |
| prod backend          | prod mysql           | ✅     |
| dev pods              | prod mysql           | ❌     |
| dev pods              | prod backend         | ❌     |
| monitoring prometheus | prod backend metrics | ✅     |
| random prod pod       | prod mysql           | ❌     |
| all pods DNS access   | kube-dns             | ✅     |

---

# Step 1: Create Namespaces

```bash
kubectl create ns dev
kubectl create ns prod
kubectl create ns monitoring
```

Label them:

```bash
kubectl label ns dev env=dev
kubectl label ns prod env=prod
kubectl label ns monitoring team=ops
```

---

# Step 2: Deploy Workloads

## Prod Frontend

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: prod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
```

---

## Prod Backend

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: prod
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args: ["-text=backend","-listen=:8080"]
```

---

## Prod MySQL

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: prod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: root123
```

---

## Monitoring Prometheus

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: prometheus
  namespace: monitoring
  labels:
    app: prometheus
spec:
  containers:
  - name: prom
    image: busybox
    command: ["sleep","3600"]
```

---

## Dev Test Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: devbox
  namespace: dev
  labels:
    app: tester
spec:
  containers:
  - name: box
    image: busybox
    command: ["sleep","3600"]
```

Apply all files.

---

# Step 3: Default Deny in Prod Namespace

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: prod
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

This means no inbound traffic to any prod pod unless explicitly allowed.

---

# Step 4: Allow Frontend → Backend

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-fe-be
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - port: 8080
```

---

# Step 5: Allow Backend → MySQL

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-be-db
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: mysql
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend
    ports:
    - port: 3306
```

---

# Step 6: Allow Monitoring Namespace → Backend Metrics

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-monitoring
  namespace: prod
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: ops
    ports:
    - port: 8080
```

---

# Step 7: Allow DNS Egress (Important)

Without this many apps fail DNS resolution.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: prod
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```

---

# Validation Tests

## Dev pod to Prod DB

```bash
kubectl exec -it devbox -n dev -- sh
nc -zv mysql.prod.svc.cluster.local 3306
```

❌ Blocked

---

## Frontend to Backend

```bash
kubectl exec -it deploy/frontend -n prod -- sh
wget -qO- backend:8080
```

✅ Allowed

---

## Backend to MySQL

```bash
kubectl exec -it deploy/backend -n prod -- sh
nc -zv mysql 3306
```

✅ Allowed

---

## Prometheus to Backend

```bash
kubectl exec -it prometheus -n monitoring -- sh
wget -qO- backend.prod.svc.cluster.local:8080
```

✅ Allowed

---

# Enterprise Lessons

## 1. Namespace Isolation

`dev` cannot touch `prod`

## 2. Least Privilege

Only required paths opened.

## 3. Monitoring Exceptions

Observability still works.

## 4. DNS Often Forgotten

Many teams break apps here.

---

# CKA Exam Shortcut Thinking

When asked NetworkPolicy:

1. Default deny first
2. Add only required flows
3. Check labels carefully
4. Test with busybox pod

---

# Real Interview Talking Point

“Implemented zero-trust Kubernetes east-west traffic controls using NetworkPolicies, namespace isolation, monitoring exceptions, and least-privilege service communication.”

---

