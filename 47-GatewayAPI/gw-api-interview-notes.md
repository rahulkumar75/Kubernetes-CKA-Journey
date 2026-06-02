# Kubernetes Gateway API — Complete CKA/Interview Notes

## What is Gateway API?

Gateway API is the **next-generation networking API** in Kubernetes designed to improve and replace many limitations of traditional Ingress.

It provides:

* Better traffic routing
* More flexibility
* Role separation
* Extensibility
* Multi-team support

It is developed by the Kubernetes SIG Network community.

---

# Why Gateway API was introduced?

Traditional Ingress had problems:

* Limited routing features
* Controller-specific annotations
* Difficult TCP/UDP support
* No clear separation between infra team and app team
* Hard to extend

Gateway API solves these issues.

---

# Main Components of Gateway API

## 1. GatewayClass

Defines:

> Which controller will manage the Gateway

Similar to:

* `StorageClass`
* `IngressClass`

Example:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: gateway.nginx.org/nginx-gateway-controller
```

---

## 2. Gateway

Represents:

> Actual load balancer / entry point into cluster

Responsible for:

* Listener configuration
* Ports
* Protocols
* TLS

Example:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: web-gateway
spec:
  gatewayClassName: nginx
  listeners:
  - name: http
    protocol: HTTP
    port: 80
```

---

## 3. HTTPRoute

Defines:

> Traffic routing rules

Responsible for:

* Path-based routing
* Header matching
* Traffic splitting
* Backend selection

Example:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: app-route
spec:
  parentRefs:
  - name: web-gateway

  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /app

    backendRefs:
    - name: my-service
      port: 80
```

---

# Architecture Flow

```text
Client Request
      ↓
Gateway
      ↓
HTTPRoute
      ↓
Service
      ↓
Pods
```

---

# Important Gateway API Resources

| Resource     | Purpose              |
| ------------ | -------------------- |
| GatewayClass | Select controller    |
| Gateway      | Entry point          |
| HTTPRoute    | HTTP traffic routing |
| TCPRoute     | TCP traffic          |
| UDPRoute     | UDP traffic          |
| TLSRoute     | TLS passthrough      |
| GRPCRoute    | gRPC routing         |

---

# Gateway API vs Ingress

| Feature           | Ingress       | Gateway API       |
| ----------------- | ------------- | ----------------- |
| Extensible        | Limited       | Highly extensible |
| TCP/UDP Support   | Poor          | Native            |
| Traffic Splitting | Limited       | Built-in          |
| Role Separation   | Weak          | Strong            |
| Header Matching   | Limited       | Advanced          |
| Multiple Teams    | Difficult     | Easy              |
| Standardization   | Less flexible | Better design     |

---

# Role Separation (Very Important)

Gateway API separates responsibilities.

## Infra Team

Manages:

* GatewayClass
* Gateway
* TLS
* Load balancer

## App Team

Manages:

* HTTPRoute
* Application routing

This is a major improvement.

---

# Traffic Splitting Example

Can split traffic like:

* 90% → v1
* 10% → v2

Useful for:

* Canary deployments
* Blue-Green deployments

Example:

```yaml
rules:
- backendRefs:
  - name: app-v1
    port: 80
    weight: 90

  - name: app-v2
    port: 80
    weight: 10
```

---

# Header-Based Routing

Example:

```yaml
matches:
- headers:
  - type: Exact
    name: version
    value: beta
```

Useful for:

* Testing
* Canary users
* Feature rollout

---

# Path-Based Routing

Example:

```yaml
matches:
- path:
    type: PathPrefix
    value: /api
```

---

# TLS in Gateway API

Gateway handles TLS listeners.

Example:

```yaml
listeners:
- name: https
  protocol: HTTPS
  port: 443
  tls:
    mode: Terminate
    certificateRefs:
    - name: tls-secret
```

---

# Supported Controllers

Popular implementations:

* NGINX Gateway Controller
* Istio
* Envoy Gateway
* Kong
* Traefik
* Cilium

---

# Install Gateway API CRDs

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml
```

---

# Check Installed Resources

```bash
kubectl get crds | grep gateway
```

---

# Real-World Use Cases

## 1. Multi-team clusters

Different teams manage routes independently.

---

## 2. Canary Deployments

Split traffic between versions.

---

## 3. API Gateway

Centralized routing and security.

---

## 4. Service Mesh Integration

Works with Istio and Envoy.

---

# Important Interview Questions

## Q1. Is Gateway API replacing Ingress?

Yes, Gateway API is considered the modern evolution of Ingress.

Ingress still works, but Gateway API provides more features.

---

## Q2. Difference between Gateway and Ingress?

Ingress:

* Single resource
* Limited features

Gateway API:

* Multiple modular resources
* Better flexibility
* Better traffic control

---

## Q3. Who manages Gateway and Routes?

* Infra/Admin Team → Gateway
* App Team → HTTPRoute

---

## Q4. Does Gateway API support TCP and UDP?

Yes.
Using:

* TCPRoute
* UDPRoute

---

## Q5. What is parentRefs?

Used in Route resources to attach routes to a Gateway.

Example:

```yaml
parentRefs:
- name: web-gateway
```

---

# Simple End-to-End Example

## Step 1 — GatewayClass

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: nginx
spec:
  controllerName: gateway.nginx.org/nginx-gateway-controller
```

---

## Step 2 — Gateway

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: main-gateway
spec:
  gatewayClassName: nginx

  listeners:
  - name: http
    protocol: HTTP
    port: 80
```

---

## Step 3 — HTTPRoute

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: app-route
spec:
  parentRefs:
  - name: main-gateway

  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /

    backendRefs:
    - name: app-service
      port: 80
```

---

# Full Flow Diagram

```text
Internet User
      ↓
Load Balancer
      ↓
Gateway
      ↓
HTTPRoute
      ↓
Service
      ↓
Pod
```

---

# CKA Important Points

✅ Gateway API uses CRDs

✅ More powerful than Ingress

✅ Supports advanced routing

✅ Better RBAC separation

✅ Supports HTTP, HTTPS, TCP, UDP, gRPC

✅ GatewayClass selects controller

✅ HTTPRoute handles routing logic

✅ Gateway handles listeners

---

# Quick Revision

```text
GatewayClass → Controller

Gateway → Entry point

HTTPRoute → Routing rules

Gateway API = Modern Ingress
```
