# Kubernetes Gateway API — Mind Map
```text
                              ┌─────────────────────┐
                              │    Gateway API      │
                              │ Modern Ingress      │
                              └──────────┬──────────┘
                                         │
        ┌────────────────────────────────┼────────────────────────────────┐
        │                                │                                │
        ▼                                ▼                                ▼

 ┌───────────────┐              ┌────────────────┐              ┌─────────────────┐
 │ Core Objects  │              │ Routing Types  │              │ Key Benefits    │
 └───────┬───────┘              └───────┬────────┘              └────────┬────────┘
         │                              │                                │
         ▼                              ▼                                ▼

 GatewayClass                  HTTPRoute                    Better than Ingress
 Gateway                       TCPRoute                     Advanced Routing
 HTTPRoute                     UDPRoute                     Traffic Splitting
                               TLSRoute                     Header Matching
                               GRPCRoute                    Team Separation

```

---

# Architecture Flow

```text
Client
  │
  ▼
Gateway
  │
  ▼
HTTPRoute
  │
  ▼
Service
  │
  ▼
Pods
```

---

# Core Components

```text
GatewayClass
     │
     ▼
Selects Controller
(Envoy / NGINX / Istio)

Gateway
     │
     ▼
Entry Point
(Listeners, Ports, TLS)

HTTPRoute
     │
     ▼
Routing Rules
(Path, Host, Headers)

Service
     │
     ▼
Backend Application
```

---

# GatewayClass

```text
Purpose:
Which controller manages Gateway?

Examples:
- Envoy Gateway
- NGINX Gateway
- Istio
- Traefik
```

Example:

```yaml
gatewayClassName: eg
```

---

# Gateway

```text
Acts Like:
Load Balancer Entry Point

Defines:
- Port
- Protocol
- TLS
- Listener
```

Example:

```yaml
listeners:
- name: http
  protocol: HTTP
  port: 80
```

---

# HTTPRoute

```text
Responsible For:
- Path Routing
- Host Routing
- Header Routing
- Traffic Splitting
- URL Rewrites
```

---

# Routing Types

## Path Based

```text
/api  → backend-api
/web  → backend-web
```

Example:

```yaml
matches:
- path:
    type: PathPrefix
    value: /api
```

---

## Host Based

```text
api.example.com
        ↓
backend-api

app.example.com
        ↓
backend-app
```

Example:

```yaml
hostnames:
- "api.example.com"
```

---

## Header Based

```text
version=beta
      ↓
backend-v2

version=stable
      ↓
backend-v1
```

Example:

```yaml
headers:
- name: version
  value: beta
```

---

# Filters

```text
HTTPRoute Filters
       │
       ├── URL Rewrite
       ├── Redirect
       ├── Header Add
       ├── Header Remove
       └── Request Mirror
```

---

# URL Rewrite

```text
Client:
/get

Gateway:
/replace

Backend receives:
/replace
```

---

# Traffic Splitting

```text
HTTPRoute
      │
      ▼

 ┌──────────┐
 │backend-v1│ 90%
 └──────────┘

 ┌──────────┐
 │backend-v2│ 10%
 └──────────┘
```

Example:

```yaml
backendRefs:
- name: backend-v1
  weight: 90

- name: backend-v2
  weight: 10
```

---

# Production Use Cases

```text
Canary Deployment
90% Stable
10% New Version

Blue/Green Deployment
50% Blue
50% Green

A/B Testing
Version A
Version B

Progressive Delivery
Gradual Rollout
```

---

# Verification Flow

```text
GatewayClass
      │
      ▼
Gateway
      │
      ▼
HTTPRoute
      │
      ▼
Service
      │
      ▼
Endpoints
      │
      ▼
Pods
      │
      ▼
Curl Test
```

Commands:

```bash
kubectl get gatewayclass
kubectl get gateway
kubectl get httproute
kubectl get svc
kubectl get endpoints
kubectl get pods
```

---

# Common Debugging Map

```text
Request Fails
      │
      ▼

Gateway
      │
      ├─ PROGRAMMED=False
      ├─ GatewayClass issue
      └─ Controller missing

HTTPRoute
      │
      ├─ Accepted=False
      ├─ ResolvedRefs=False
      └─ Host mismatch

Service
      │
      ├─ Wrong Selector
      └─ No Endpoints

Pod
      │
      ├─ CrashLoopBackOff
      ├─ ImagePullBackOff
      └─ ServiceAccount Missing
```

---

# Gateway API vs Ingress

```text
Ingress
   │
   ├─ Basic Routing
   ├─ Limited Features
   └─ Annotation Heavy

Gateway API
   │
   ├─ Advanced Routing
   ├─ TCP/UDP Support
   ├─ Traffic Splitting
   ├─ Header Matching
   └─ Better Design
```

---

# 0–3 Year Interview Revision

```text
GatewayClass → Controller

Gateway → Entry Point

HTTPRoute → Routing Rules

backendRefs → Backend Services

Filters → Rewrite / Redirect

weight → Traffic Split

Accepted=True
ResolvedRefs=True
PROGRAMMED=True
```

# One-Line Memory Trick

```text
GatewayClass → Gateway → HTTPRoute → Service → Endpoints → Pods
                      ↓
            Filters + Traffic Splitting
```

This single mind map covers about **90% of Gateway API concepts needed for CKA practice, troubleshooting, and 0–3 year DevOps/Kubernetes interviews**.
