# 🌐 Kubernetes Ingress — CKA Revision Mind Map

```text
                                ┌─────────────────────┐
                                │      INGRESS        │
                                └──────────┬──────────┘
                                           │
             ┌─────────────────────────────┼─────────────────────────────┐
             │                             │                             │
             ▼                             ▼                             ▼
    ┌────────────────┐          ┌─────────────────┐          ┌─────────────────┐
    │ WHAT IS IT?    │          │ CONTROLLER      │          │ WHY USE IT?     │
    └────────────────┘          └─────────────────┘          └─────────────────┘
             │                             │                             │
             ├─ L7 Routing                 ├─ NGINX Ingress              ├─ Single Entry Point
             ├─ HTTP/HTTPS Only            ├─ Traefik                    ├─ Path Routing
             ├─ External Access            ├─ HAProxy                    ├─ Host Routing
             └─ Rules for Traffic          └─ Required for Ingress       └─ TLS Termination

                                           │
                                           ▼
                              ┌───────────────────────┐
                              │ REQUEST FLOW          │
                              └───────────────────────┘
                                           │
                                           ▼
      User → DNS → Ingress Controller → Service → Pod

                                           │
             ┌─────────────────────────────┼─────────────────────────────┐
             │                             │                             │
             ▼                             ▼                             ▼

 ┌────────────────────┐      ┌───────────────────┐       ┌──────────────────┐
 │ HOST-BASED ROUTING │      │ PATH-BASED ROUTE  │       │ TLS / HTTPS      │
 └────────────────────┘      └───────────────────┘       └──────────────────┘
             │                           │                            │
             │ example.com               │ /api                      │ Secret
             │ app.example.com           │ /admin                    │ Certificate
             │ api.example.com           │ /app                      │ HTTPS
             ▼                           ▼                            ▼

      Route by Domain            Route by URL Path         SSL Termination

                                           │
                                           ▼
                              ┌───────────────────────┐
                              │ INGRESS COMPONENTS    │
                              └───────────────────────┘
                                           │
              ┌────────────────────────────┼───────────────────────────┐
              │                            │                           │
              ▼                            ▼                           ▼

       ingressClassName             rules                    backend
             │                        │                         │
             │ nginx                  │ host/path               │ service
             │                        │                         │ port

                                           │
                                           ▼
                            ┌────────────────────────┐
                            │ COMMON COMMANDS        │
                            └────────────────────────┘
                                           │
        kubectl get ingress
        kubectl describe ingress <name>
        kubectl get ingressclass
        kubectl get svc -n ingress-nginx
        kubectl logs -n ingress-nginx <pod>

                                           │
                                           ▼
                            ┌────────────────────────┐
                            │ TROUBLESHOOTING        │
                            └────────────────────────┘
                                           │

      404 Not Found
      ├─ Host mismatch
      ├─ Path mismatch
      └─ Rule not matched

      502 Bad Gateway
      ├─ Service issue
      ├─ Endpoint missing
      └─ Pod not reachable

      ADDRESS Empty
      ├─ No LoadBalancer
      ├─ Local Cluster
      └─ Use NodePort/Port-Forward

      Ingress Not Working
      ├─ Controller Missing
      ├─ Wrong ingressClassName
      ├─ Wrong Service Name
      └─ Wrong Port

                                           │
                                           ▼
                            ┌────────────────────────┐
                            │ LOCAL CLUSTER NOTES    │
                            └────────────────────────┘
                                           │

      KIND / Docker Desktop
      ├─ LoadBalancer → <pending>
      ├─ Use NodePort
      ├─ Use Port Forward
      └─ Add /etc/hosts Entry

      Example:
      127.0.0.1 example.com

                                           │
                                           ▼
                            ┌────────────────────────┐
                            │ CKA EXAM FOCUS         │
                            └────────────────────────┘
                                           │

      ✓ Create Ingress
      ✓ Path Routing
      ✓ Host Routing
      ✓ Debug 404 Errors
      ✓ Check Endpoints
      ✓ Verify IngressClass
      ✓ Use kubectl describe ingress
      ✓ Understand Controller Requirement
```

## 🚀 30-Second Revision

```text
Ingress = L7 HTTP/HTTPS Router

Ingress Resource = Rules
Ingress Controller = Actual Engine (NGINX)

Flow:
User → Ingress → Service → Pod

Routing:
✓ Host-based
✓ Path-based

Common Errors:
404 → Host/Path mismatch
502 → Service/Pod issue
ADDRESS Empty → No LoadBalancer

Debug:
kubectl get ingress
kubectl describe ingress
kubectl get endpoints
kubectl logs -n ingress-nginx <pod>
```

This mind map covers about **90–95% of Ingress concepts needed for CKA and 0–3 years DevOps interviews**.
