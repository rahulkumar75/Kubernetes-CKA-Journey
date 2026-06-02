# Kubernetes NetworkPolicy Hands-on Lab

## 3-Tier App: Frontend + Backend + Database

This lab gives you a **real enterprise-style setup** with:

* **frontend** → web UI pods
* **backend** → API pods
* **database** → MySQL pod

And we secure traffic using **NetworkPolicy**.

---

# Architecture

```text
User
 ↓
Frontend Pods (nginx)
 ↓
Backend Pods (api)
 ↓
MySQL DB
```

## Allowed Traffic

| Source     | Destination | Port | Allow |
| ---------- | ----------- | ---- | ----- |
| Frontend   | Backend     | 8080 | ✅     |
| Backend    | MySQL       | 3306 | ✅     |
| Frontend   | MySQL       | 3306 | ❌     |
| Random Pod | Backend     | 8080 | ❌     |
| Random Pod | DB          | 3306 | ❌     |

---

# Step 1: Create Namespace

```bash
kubectl create ns ecommerce
```

---

# Step 2: Deploy Frontend

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: ecommerce
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        tier: web
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

```bash
kubectl apply -f frontend.yaml
```

---

# Step 3: Deploy Backend

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
  namespace: ecommerce
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
        tier: api
    spec:
      containers:
      - name: backend
        image: hashicorp/http-echo
        args:
        - "-text=backend-api"
        - "-listen=:8080"
        ports:
        - containerPort: 8080
```

```bash
kubectl apply -f backend.yaml
```

---

# Step 4: Deploy MySQL DB

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: ecommerce
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
        tier: db
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: root123
        ports:
        - containerPort: 3306
```

```bash
kubectl apply -f mysql.yaml
```

---

# Step 5: Create Services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: ecommerce
spec:
  selector:
    app: frontend
  ports:
  - port: 80

---
apiVersion: v1
kind: Service
metadata:
  name: backend
  namespace: ecommerce
spec:
  selector:
    app: backend
  ports:
  - port: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: mysql
  namespace: ecommerce
spec:
  selector:
    app: mysql
  ports:
  - port: 3306
```

```bash
kubectl apply -f svc.yaml
```

---

# Step 6: Verify Pods

```bash
kubectl get pods -o wide -n ecommerce
```

You should see pods spread across your 3 nodes.

---

# Step 7: Test Before Policies

Create temp pod:

```bash
kubectl run test --rm -it --image=busybox -n ecommerce -- sh
```

Inside:

```bash
wget -qO- backend:8080
nc -zv mysql 3306
```

Everything works now (open traffic).

---

# Step 8: Apply Default Deny for Backend + DB

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-backend-db
  namespace: ecommerce
spec:
  podSelector:
    matchExpressions:
    - key: app
      operator: In
      values:
      - backend
      - mysql
  policyTypes:
  - Ingress
```

```bash
kubectl apply -f deny.yaml
```

Now backend and DB blocked.

---

# Step 9: Allow Frontend → Backend

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-backend
  namespace: ecommerce
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
    - protocol: TCP
      port: 8080
```

```bash
kubectl apply -f frontend-backend.yaml
```

---

# Step 10: Allow Backend → MySQL

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-backend-db
  namespace: ecommerce
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
    - protocol: TCP
      port: 3306
```

```bash
kubectl apply -f backend-db.yaml
```

---

# Final Security Result

| Source     | Backend  | MySQL |
| ---------- | -------- | ----- |
| Frontend   | ✅        | ❌     |
| Backend    | internal | ✅     |
| Random Pod | ❌        | ❌     |

---

# Step 11: Testing

## Frontend Pod → Backend

```bash
kubectl exec -it deploy/frontend -n ecommerce -- sh
wget -qO- backend:8080
```

✅ Works

---

## Frontend Pod → MySQL

```bash
nc -zv mysql 3306
```

❌ Blocked

---

## Backend Pod → MySQL

```bash
kubectl exec -it deploy/backend -n ecommerce -- sh
nc -zv mysql 3306
```

✅ Works

---

## Random Pod → Backend

```bash
kubectl run hacker --rm -it --image=busybox -n ecommerce -- sh
wget -qO- backend:8080
```

❌ Blocked

---

# Enterprise Logic

This is how real companies secure apps:

```text
Frontend can call API
API can call DB
No one else can move laterally
```

If frontend hacked → attacker still cannot access DB directly.

---

# Important Note

Need CNI with NetworkPolicy support:

* Calico
* Cilium
* Antrea

---

# CKA Exam Tip

Remember:

```bash
kubectl get netpol -n ecommerce
kubectl describe netpol -n ecommerce
```

---