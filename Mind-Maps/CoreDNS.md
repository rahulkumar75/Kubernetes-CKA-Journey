# ☸️ CoreDNS in Kubernetes – Mind Map for 0–3 Year DevOps / CKA Interviews

```text
                                   ┌──────────────────────┐
                                   │      CoreDNS        │
                                   │ Kubernetes DNS      │
                                   └──────────┬──────────┘
                                              │
          ┌───────────────────────────────────┼───────────────────────────────────┐
          │                                   │                                   │
          ▼                                   ▼                                   ▼

 ┌─────────────────┐                ┌─────────────────┐                ┌─────────────────┐
 │     PURPOSE     │                │  DEPLOYMENT     │                │   COMPONENTS    │
 └────────┬────────┘                └────────┬────────┘                └────────┬────────┘
          │                                  │                                  │
          ▼                                  ▼                                  ▼

 • Service Discovery             • Runs as Deployment              • CoreDNS Pods
 • Name Resolution               • kube-system Namespace           • Service
 • Pod → Service Lookup          • Usually 2 Replicas              • ConfigMap
 • Internal Cluster DNS          • Highly Available                • Corefile


                                              │
                                              ▼

                              ┌─────────────────────────┐
                              │      HOW IT WORKS       │
                              └───────────┬─────────────┘
                                          │
                                          ▼

                 Pod Requests DNS Lookup (my-service)
                                │
                                ▼
                       /etc/resolv.conf
                                │
                                ▼
                         Cluster DNS IP
                     (e.g. 10.96.0.10)
                                │
                                ▼
                            CoreDNS
                                │
                 ┌──────────────┴──────────────┐
                 │                             │
                 ▼                             ▼

       Internal Service Lookup       External DNS Lookup
       (Kubernetes API)              (Forward to Upstream DNS)

                 │                             │
                 ▼                             ▼

       Returns Service IP          Returns Internet IP
       (ClusterIP)                 (google.com etc.)


          ┌───────────────────────────────────┼───────────────────────────────────┐
          │                                   │                                   │
          ▼                                   ▼                                   ▼

 ┌─────────────────┐                ┌─────────────────┐                ┌─────────────────┐
 │ DNS RECORDS     │                │ SERVICE NAMES   │                │ CONFIGURATION   │
 └────────┬────────┘                └────────┬────────┘                └────────┬────────┘
          │                                  │                                  │
          ▼                                  ▼                                  ▼

 • A Record                     • service-name                 • Corefile
 • SRV Record                   • service.namespace            • Plugins
 • Pod Records                  • service.namespace.svc        • Forward
 • Service Records              • svc.cluster.local            • Cache
                                                                  • Health


                                              │
                                              ▼

                              ┌─────────────────────────┐
                              │     K8s DNS FLOW        │
                              └───────────┬─────────────┘
                                          │
                                          ▼

     Frontend Pod
            │
            ▼
     backend-service
            │
            ▼
     CoreDNS Query
            │
            ▼
     Kubernetes API
            │
            ▼
     Finds ClusterIP
            │
            ▼
     Returns IP
            │
            ▼
     Pod Connects Successfully


          ┌───────────────────────────────────┼───────────────────────────────────┐
          │                                   │                                   │
          ▼                                   ▼                                   ▼

 ┌─────────────────┐                ┌─────────────────┐                ┌─────────────────┐
 │ TROUBLESHOOTING │                │ COMMON ISSUES   │                │ INTERVIEW Q&A   │
 └────────┬────────┘                └────────┬────────┘                └────────┬────────┘
          │                                  │                                  │
          ▼                                  ▼                                  ▼

 • nslookup service            • CoreDNS Crash                 • What is CoreDNS?
 • dig service                 • Wrong Corefile                • Why needed?
 • check pods                  • DNS Timeout                   • How service discovery
 • check service               • Network Policy Block            works?
 • check logs                  • Missing Endpoint              • Where does CoreDNS run?
                                                                    • How to debug DNS?


                                              │
                                              ▼

                                IMPORTANT COMMANDS

kubectl get pods -n kube-system

kubectl get svc -n kube-system

kubectl logs -n kube-system deployment/coredns

kubectl exec -it busybox -- nslookup kubernetes.default

kubectl get configmap coredns -n kube-system

kubectl describe pod <pod-name>
```

---

# 🎯 Interview Quick Revision (30 Seconds)

```text
CoreDNS = Kubernetes DNS Server

Purpose:
Pod → Service Name Resolution

Flow:
Pod
 ↓
/etc/resolv.conf
 ↓
CoreDNS
 ↓
Kubernetes API
 ↓
Service IP Returned

Important FQDN:
my-service.default.svc.cluster.local

Runs:
Deployment in kube-system namespace

Debug:
kubectl get pods -n kube-system
kubectl logs deployment/coredns -n kube-system
nslookup my-service

If CoreDNS Fails:
❌ Service discovery fails
❌ Pods cannot resolve names
❌ Microservice communication breaks
```

# 🔥 Most Asked Interview Questions (0–3 Years)

### Level 1

1. What is CoreDNS?
2. Why is CoreDNS needed in Kubernetes?
3. Where does CoreDNS run?

### Level 2

4. How does a Pod resolve a Service name?
5. What is `svc.cluster.local`?
6. What is stored in `/etc/resolv.conf`?

### Level 3

7. How do you troubleshoot DNS issues in Kubernetes?
8. What happens if CoreDNS goes down?
9. How does CoreDNS know Service IPs?
10. Difference between DNS and CoreDNS?

---

### CKA Exam Focus ⭐⭐⭐⭐⭐

```text
Check CoreDNS:
kubectl get pods -n kube-system

Test DNS:
kubectl exec -it busybox -- nslookup kubernetes.default

Check Config:
kubectl get cm coredns -n kube-system -o yaml

Check Logs:
kubectl logs -n kube-system deployment/coredns
```

This mind map covers about **95% of CoreDNS questions asked in CKA, Kubernetes Admin, Junior DevOps, and Platform Engineer interviews**.
