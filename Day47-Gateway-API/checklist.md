# Gateway API Traffic Splitting / Weighted Routing Checklist

---

# 1. Gateway API CRDs Installed

Verify:

```bash id="v4y1ah"
kubectl get crds | grep gateway
```

Expected:

```text id="e8m0pn"
gatewayclasses.gateway.networking.k8s.io
gateways.gateway.networking.k8s.io
httproutes.gateway.networking.k8s.io
```

---

# 2. Gateway Controller Running

Verify:

```bash id="r5t9kx"
kubectl get pods -A
```

Expected:

```text id="w2p6zf"
envoy-gateway-system
```

OR:

* nginx gateway
* istio
* traefik

---

# 3. GatewayClass Exists

Verify:

```bash id="x0c7dy"
kubectl get gatewayclass
```

Expected:

```text id="n1v5qs"
eg
```

---

# 4. Gateway Created Successfully

Verify:

```bash id="j7u3mw"
kubectl get gateway
```

Expected:

```text id="b4l8yr"
PROGRAMMED=True
```

If False:

```bash id="y9n2qe"
kubectl describe gateway eg
```

---

# 5. Backend Deployments Running

Verify:

```bash id="f6m1tp"
kubectl get deploy
```

Expected:

```text id="q8k4av"
backend
backend-2
```

---

# 6. Backend Pods Running

Verify:

```bash id="t3w8cs"
kubectl get pods
```

Expected:

```text id="m0x7pl"
Running
```

---

# 7. Services Created

Verify:

```bash id="c9r2jh"
kubectl get svc
```

Expected:

```text id="s4d6kw"
backend
backend-2
```

---

# 8. Service Selectors Match Pod Labels

Verify:

```bash id="h1p9vx"
kubectl get pods --show-labels
```

AND:

```bash id="l7y4qe"
kubectl get svc backend -o yaml
kubectl get svc backend-2 -o yaml
```

Check:

```text id="u2n5jm"
selector == pod labels
```

---

# 9. Endpoints Exist

Very important.

Verify:

```bash id="k5w8tz"
kubectl get endpoints
```

Expected:

```text id="d0q4cb"
backend      <pod-ip>:3000
backend-2    <pod-ip>:3000
```

If `<none>`:

* selector mismatch
* pod not ready

---

# 10. HTTPRoute Accepted

Verify:

```bash id="v8r6nx"
kubectl get httproute
```

Then:

```bash id="e3k1ms"
kubectl describe httproute http-headers
```

Expected:

```text id="y5c9tp"
Accepted=True
ResolvedRefs=True
```

---

# 11. backendRefs Correct

Verify:

* service names correct
* ports correct
* weights configured

Example:

```yaml id="q7m2lf"
backendRefs:
- name: backend
  port: 3000
  weight: 50

- name: backend-2
  port: 3000
  weight: 50
```

---

# 12. Hostname Matches Curl Header

If HTTPRoute contains:

```yaml id="p3x8wd"
hostnames:
- "backends.example"
```

Then curl MUST use:

```bash id="z6t1kh"
curl -H "Host: backends.example" http://localhost:8080
```

Otherwise:

```text id="w9m4qf"
404 Not Found
```

---

# 13. Envoy Service Exposed

Verify:

```bash id="n2j7cy"
kubectl get svc -n envoy-gateway-system
```

---

# 14. Port Forward Running

Example:

```bash id="g4q9sb"
kubectl port-forward svc/<envoy-service> 8080:80 -n envoy-gateway-system
```

Keep terminal open.

---

# 15. Verify Traffic Splitting

Test:

```bash id="r1v6pk"
for i in {1..10}
do
  curl -s -H "Host: backends.example" http://localhost:8080
done
```

---

# 16. Better Verification Using Pod Name

Add env vars:

```yaml id="t8y2mj"
env:
- name: POD_NAME
  valueFrom:
    fieldRef:
      fieldPath: metadata.name
```

Then:

```bash id="u5k7nr"
curl -s -H "Host: backends.example" http://localhost:8080 | jq '.pod'
```

Expected:

```text id="f3p8lx"
backend-xxxxx
backend-2-xxxxx
```

---

# 17. Common Problems Checklist

| Problem               | Reason                        |
| --------------------- | ----------------------------- |
| PROGRAMMED=False      | GatewayClass/controller issue |
| Accepted=False        | Invalid route                 |
| ResolvedRefs=False    | Service missing               |
| 404                   | Host mismatch                 |
| connection refused    | Port-forward missing          |
| empty endpoints       | Selector mismatch             |
| pod not created       | ServiceAccount/image issue    |
| traffic not splitting | only one backend healthy      |

---

# 18. Real Production Use Cases

```text id="k1r5wv"
90% → stable
10% → canary
```

```text id="q4m8tz"
Blue/Green deployment
```

```text id="y7n2pc"
A/B testing
```

```text id="d6w9kl"
Progressive rollout
```

---

# Final Debugging Flow

```text id="b8x3qh"
GatewayClass
   ↓
Gateway
   ↓
HTTPRoute
   ↓
Service
   ↓
Endpoints
   ↓
Pods
   ↓
Curl Test
```

This flow solves most Gateway API traffic issues.
