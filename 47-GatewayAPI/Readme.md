# Kubernetes Gateway API

> Study notes for revision and implementation.

---

## 1. The Core Problem

Kubernetes solves container orchestration. Companies use it to deploy microservices like Login, Product, Payment, Order, and Cart.

### East–West Traffic ✅

- Service-to-service communication **inside** the cluster.
- Kubernetes handles this natively with its in-cluster network.
- This was **never a problem**.

### North–South Traffic ⚠️

- External users trying to reach pods **inside** the cluster.
- Kubernetes has no native load balancer.
- This was the **major challenge**.

---

## 2. First Solution: Ingress Controller

To solve North–South traffic, Kubernetes introduced the **Ingress Controller**.

Providers: Nginx, HAProxy, AWS, etc.

### How it works

1. **Cluster admin** installs the Ingress controller.
2. **DevOps engineer** creates an Ingress resource.
3. The controller watches Ingress resources and creates a **Load Balancer**.
4. The Load Balancer is public-facing. External users talk to it, and it routes traffic to the right microservice.

---

## 3. Drawbacks of Ingress

### Drawback 1: Limited Features

Ingress does **not** natively support these production-level features:

| Feature | Supported by Ingress? |
|---|---|
| Rate Limiting | ❌ |
| WAF (Web Application Firewall) | ❌ |
| HTTPS | ❌ |
| Canary Deployments | ❌ |
| DDoS Protection | ❌ |
| URL Rewrite | ❌ |

**Workaround:** Controller providers added **annotations** to fill the gap.

**Problem with annotations:**
- They are in JSON format — hard to read.
- A simple Ingress resource can have 30–40 annotations.
- Annotations differ from controller to controller.
- Managing them becomes complex over time.

### Drawback 2: No Role Separation

- Platform engineers and DevOps engineers both edit the **same** Ingress file.
- Anyone can accidentally change the `ingressClassName` field.
- There is no way to enforce who can edit what.

---

## 4. New Solution: Gateway API

Gateway API solves the same North–South traffic problem — but it **fixes both Ingress drawbacks**.

### Key Idea: Split Into Three Resources

Instead of one resource, the Kubernetes community created **three separate CRDs**:

| Resource | Owner | Purpose |
|---|---|---|
| `GatewayClass` | Platform Engineer | Defines which controller to use |
| `Gateway` | DevOps Engineer | Defines which GatewayClass to use and which port to listen on |
| `HTTPRoute` | DevOps Engineer | Defines backend service, port, path, and advanced routing rules |

> All three are **Custom Resources (CRDs)** — an added advantage.

### How Each Resource Works

#### GatewayClass
- Created when the controller is installed.
- Acts like the Ingress class — but in its own dedicated resource.
- Example: points to the `envoy-proxy` controller.

#### Gateway
- Specifies which `GatewayClass` to use.
- Configures the listener — for example, listen on port 80.

#### HTTPRoute
- The most important resource.
- Defines backend service name and port.
- Supports hostname filtering (e.g., only accept `www.example.com`).
- Supports advanced features: traffic splitting, URL rewrite, WAF, etc.

---

## 5. Ingress vs Gateway API

| Area | Ingress | Gateway API |
|---|---|---|
| Feature support | Limited spec; needs 30–40 annotations | Rate limiting, canary, URL rewrite built in |
| Portability | Annotations differ per controller | Standard spec across controllers |
| Role separation | One file; anyone can change anything | Separate resources per role |
| Resource type | Single built-in resource | Three CRDs |

---

## 6. Implementation Demo

### Step 0 — Install

```bash
# Install Gateway API CRDs
kubectl apply -f gateway-api-crds.yaml

# Install Envoy Gateway controller
# (Beginner-friendly; waits for Gateway and HTTPRoute resources)
kubectl apply -f envoy-gateway.yaml

# Verify controller is running
kubectl get pods -n envoy-gateway-system
```

> **Why Envoy?** Local `kind` clusters do not support actual load balancer service types. Envoy acts as a reverse proxy instead.

---

### Step 1 — Deploy the App

```bash
kubectl apply -f deploy.yaml
kubectl apply -f svc.yaml
kubectl apply -f svc_account.yaml
```

> **Note:** Always create a dedicated service account. If you don't, the default service account is used.

The service is `ClusterIP` on port `3000`. It is only reachable inside the cluster at this point.

**Verify inside the cluster:**
```bash
kubectl exec -it <pod-name> -- curl http://<service-name>:3000
```

---

### Step 2 — Expose to the Outside World

**File 1: `gatewayclass.yaml`**

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: eg
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
```

> This tells any Gateway using this class to use the Envoy proxy controller.

---

**File 2: `gateway.yaml`**

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: eg
spec:
  gatewayClassName: eg
  listeners:
    - name: http
      protocol: HTTP
      port: 80
```

> The load balancer listens on port 80. Without `gatewayClassName`, the Gateway does not know which controller to use.

---

**File 3: `httproutes.yaml`**

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: my-app
spec:
  parentRefs:
    - name: eg
  hostnames:
    - "www.example.com"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: my-service
          port: 3000
```

> Request flow: External user → Load Balancer → HTTPRoute → Service on port 3000.
> Only requests with hostname `www.example.com` are accepted.

---

**Verify and test:**

```bash
# Check pods in Envoy namespace
kubectl get pods -n envoy-gateway-system

# Port-forward to test locally
kubectl port-forward svc/envoy-eg-http -n envoy-gateway-system 9090:80

# Send a test request
curl -H "Host: example.com" http://localhost:9090
```

---

## 7. Advanced Feature: URL Rewrite

URL rewrite works like **call forwarding** on a phone. The user sends a request to one path, and the load balancer changes it to a different path before forwarding.

**Example:** User requests `/get` → Load Balancer rewrites to `/replace`

Only the `HTTPRoute` file needs to change. Add a `URLRewrite` filter:

```yaml
rules:
  - matches:
      - path:
          type: PathPrefix
          value: /get
    filters:
      - type: URLRewrite
        urlRewrite:
          path:
            type: ReplaceFullPath
            replaceFullPath: /replace
    backendRefs:
      - name: my-service
        port: 3000
```

---

## 8. Advanced Feature: Traffic Splitting

**Use case:** You deploy a new version of your app and want to split traffic between old and new.

Only the `HTTPRoute` file needs to change. Define two backend refs:

### 50 / 50 Split

```yaml
rules:
  - backendRefs:
      - name: backend-v1
        port: 3000
        weight: 1
      - name: backend-v2
        port: 3000
        weight: 1
```

### 80 / 20 Split (Weight-Based)

```yaml
rules:
  - backendRefs:
      - name: backend-v1
        port: 3000
        weight: 8
      - name: backend-v2
        port: 3000
        weight: 2
```

> `weight: 8` and `weight: 2` means 80% goes to `backend-v1` and 20% goes to `backend-v2`.

---

## 9. Quick Reference

### Traffic Flow Summary

```
External User
     ↓
Load Balancer (created by Gateway)
     ↓
HTTPRoute (matches hostname + path)
     ↓
Service (ClusterIP, port 3000)
     ↓
Pod
```

### Files Needed for a Full Setup

| File | Resource | Who creates it |
|---|---|---|
| `gatewayclass.yaml` | GatewayClass | Platform Engineer |
| `gateway.yaml` | Gateway | DevOps Engineer |
| `httproutes.yaml` | HTTPRoute | DevOps Engineer |
| `deploy.yaml` | Deployment | DevOps Engineer |
| `svc.yaml` | Service (ClusterIP) | DevOps Engineer |
| `svc_account.yaml` | ServiceAccount | DevOps Engineer |

### Common Commands

```bash
# Apply all resources
kubectl apply -f <filename>.yaml

# Check CRDs
kubectl get crds | grep gateway

# Check Gateway status
kubectl get gateway

# Check HTTPRoute status
kubectl get httproute

# Check Envoy pods
kubectl get pods -n envoy-gateway-system

# Port-forward for local testing
kubectl port-forward svc/<envoy-service> -n envoy-gateway-system 9090:80
```

---

*End of notes.*