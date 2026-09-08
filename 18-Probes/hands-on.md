# 🧪 8. Hands-on — Liveness Probe with Exec

## Step 1 — Create the Pod

Create `liveness-exec.yaml`:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: liveness-exec
  labels:
    test: liveness

spec:
  containers:
    - name: liveness
      image: busybox:1.36

      args:
        - /bin/sh
        - -c
        - |
          touch /tmp/healthy
          sleep 30
          rm -f /tmp/healthy
          sleep 600

      livenessProbe:
        exec:
          command:
            - cat
            - /tmp/healthy

        initialDelaySeconds: 5
        periodSeconds: 5
```

Apply:

```bash
kubectl apply -f liveness-exec.yaml
```

---

## Step 2 — Observe the Pod

```bash
kubectl get pod liveness-exec -w
```

Initially:

```text
Running
```

For the first 30 seconds:

```text
/tmp/healthy exists
       ↓
cat succeeds
       ↓
Liveness = Success
```

After 30 seconds:

```text
/tmp/healthy removed
       ↓
cat fails
       ↓
Liveness = Failure
       ↓
Container restarted
```

Check:

```bash
kubectl get pod liveness-exec
```

Look at the restart count:

```bash
kubectl get pod liveness-exec
```

You should see the `RESTARTS` count increase.

---

## Step 3 — Troubleshoot

```bash
kubectl describe pod liveness-exec
```

Check the **Events** section.

You can also inspect:

```bash
kubectl get pod liveness-exec -o wide
```

### 🧠 What did we learn?

```text
Exec command succeeds
→ Healthy

Exec command fails repeatedly
→ Liveness failure

Liveness failure
→ Container restart
```

---

# 🧪 9. HTTP Liveness Probe

Create `liveness-http.yaml`:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: liveness-http

spec:
  containers:
    - name: nginx
      image: nginx:latest

      ports:
        - containerPort: 80

      livenessProbe:
        httpGet:
          path: /
          port: 80

        initialDelaySeconds: 5
        periodSeconds: 5
```

Apply:

```bash
kubectl apply -f liveness-http.yaml
```

Check:

```bash
kubectl get pod liveness-http
```

Because nginx serves `/`:

```text
GET /
  ↓
HTTP 200
  ↓
Liveness succeeds
```

---

# 🧪 10. Test HTTP Probe Failure

Change:

```yaml
path: /notfound
```

Apply the updated Pod configuration by recreating the Pod:

```bash
kubectl delete pod liveness-http
kubectl apply -f liveness-http.yaml
```

Now:

```text
GET /notfound
      ↓
HTTP 404
      ↓
Probe failure
      ↓
Repeated failures
      ↓
Container restarted
```

Check:

```bash
kubectl describe pod liveness-http
```

Look at Events and restart count:

```bash
kubectl get pod liveness-http
```

---

# 🧪 11. Readiness Probe Demo

Use an HTTP endpoint that intentionally fails.

Create `readiness-http.yaml`:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: readiness-http
  labels:
    app: readiness-demo

spec:
  containers:
    - name: nginx
      image: nginx:latest

      ports:
        - containerPort: 80

      readinessProbe:
        httpGet:
          path: /notfound
          port: 80

        initialDelaySeconds: 5
        periodSeconds: 5
```

Apply:

```bash
kubectl apply -f readiness-http.yaml
```

Check:

```bash
kubectl get pod readiness-http
```

You may see:

```text
READY   0/1
STATUS  Running
```

This is important:

> **The Pod can be Running while not Ready.**

The failed readiness probe does **not** restart the container.

---

# 12. See Readiness Through a Service

A Service sends traffic only to **Ready** Pods.

The cleanest way to demonstrate this is with a Deployment.

## Step 1 — Create Deployment

Create `readiness-deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: readiness-demo

spec:
  replicas: 1

  selector:
    matchLabels:
      app: readiness-demo

  template:
    metadata:
      labels:
        app: readiness-demo

    spec:
      containers:
        - name: nginx
          image: nginx:latest

          ports:
            - containerPort: 80

          readinessProbe:
            httpGet:
              path: /notfound
              port: 80

            initialDelaySeconds: 5
            periodSeconds: 5
```

Apply:

```bash
kubectl apply -f readiness-deployment.yaml
```

Check:

```bash
kubectl get pods
```

You should see:

```text
READY   STATUS
0/1     Running
```

The container is running, but the readiness probe fails because `/notfound` returns HTTP `404`.

---

## Step 2 — Create the Service

```bash
kubectl expose deployment readiness-demo \
  --name=readiness-service \
  --port=80 \
  --target-port=80
```

Check:

```bash
kubectl get svc readiness-service
```

Now check the Service endpoints:

```bash
kubectl get endpoints readiness-service
```

The Pod should **not appear as a ready endpoint** because its readiness probe is failing.

For newer Kubernetes versions, you can also check:

```bash
kubectl get endpointslice
```

---

## Step 3 — Fix the Readiness Probe

Change:

```yaml
readinessProbe:
  httpGet:
    path: /notfound
    port: 80
```

to:

```yaml
readinessProbe:
  httpGet:
    path: /
    port: 80
```

Apply the updated Deployment:

```bash
kubectl apply -f readiness-deployment.yaml
```

Watch the Pod:

```bash
kubectl get pods -w
```

Then check:

```bash
kubectl get endpoints readiness-service
```

Now the Pod should appear as a Service endpoint.

---

## Step 4 — Understand the Flow

### When Readiness Fails

```text
Pod Running
    ↓
Readiness Probe → 404
    ↓
Pod = NotReady
    ↓
Service removes Pod from endpoints
    ↓
No traffic sent to Pod
```

### When Readiness Succeeds

```text
Pod Running
    ↓
Readiness Probe → 200
    ↓
Pod = Ready
    ↓
Service adds Pod to endpoints
    ↓
Traffic can reach Pod
```

### 🧠 Important

> **Running ≠ Ready**

A Pod can be:

```text
STATUS = Running
READY  = 0/1
```

The container is running, but Kubernetes does not consider it ready to receive Service traffic.

---

## Step 5 — Verify the Pod Was Not Restarted

Check:

```bash
kubectl get pods
```

Look at:

```text
RESTARTS
```

A failed readiness probe by itself does **not** restart the container.

### 🧠 CKA Recall

```text
Readiness fails
→ Pod remains Running
→ Pod becomes NotReady
→ Service removes it from endpoints
```

---

## Cleanup

```bash
kubectl delete deployment readiness-demo
kubectl delete service readiness-service
```


### 🧠 Key Difference

```text
Liveness failure
→ Restart container

Readiness failure
→ Remove Pod from Service endpoints
```

---

# 🧪 13. TCP Probe

A TCP probe checks whether a TCP connection can be established.

Example using nginx:

```yaml
livenessProbe:
  tcpSocket:
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 5
```

Complete example:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: tcp-probe

spec:
  containers:
    - name: nginx
      image: nginx:latest

      ports:
        - containerPort: 80

      livenessProbe:
        tcpSocket:
          port: 80

        initialDelaySeconds: 5
        periodSeconds: 5
```

Apply:

```bash
kubectl apply -f tcp-probe.yaml
```

Because nginx listens on port `80`:

```text
TCP connection → port 80
        ↓
Connection succeeds
        ↓
Probe succeeds
```

If you configure a port where nothing is listening:

```yaml
tcpSocket:
  port: 8080
```

the probe fails.

---



