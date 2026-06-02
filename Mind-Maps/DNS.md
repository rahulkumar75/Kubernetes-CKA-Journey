# 🌐 DNS (Domain Name System) – Mind Map
```text
                                    ┌─────────────────────┐
                                    │         DNS         │
                                    │ Domain Name System  │
                                    └──────────┬──────────┘
                                               │
        ┌──────────────────────────────────────┼──────────────────────────────────────┐
        │                                      │                                      │
        ▼                                      ▼                                      ▼

 ┌───────────────┐                    ┌────────────────┐                    ┌────────────────┐
 │   PURPOSE     │                    │ DNS COMPONENTS │                    │ DNS RECORDS    │
 └──────┬────────┘                    └───────┬────────┘                    └──────┬─────────┘
        │                                     │                                    │
        │                                     │                                    │
        ▼                                     ▼                                    ▼

 • Name → IP                    • DNS Resolver                     • A      → IPv4
 • Human Friendly               • Root Server                      • AAAA   → IPv6
 • Service Discovery            • TLD Server                       • CNAME  → Alias
 • Internet Browsing            • Authoritative NS                 • MX     → Mail
 • Kubernetes Networking        • Local Cache                      • NS     → Name Server
                                                                    • TXT    → Verification
                                                                    • PTR    → Reverse Lookup
                                                                    • SRV    → Service Discovery


                                               DNS FLOW
                                                    │
                                                    ▼

 User → Browser Cache
        ↓
 OS Cache
        ↓
 DNS Resolver
        ↓
 Root Server
        ↓
 TLD (.com/.org/.in)
        ↓
 Authoritative DNS
        ↓
 Returns IP
        ↓
 Browser Connects


        ┌──────────────────────────────────────┼──────────────────────────────────────┐
        │                                      │                                      │
        ▼                                      ▼                                      ▼

 ┌───────────────┐                    ┌────────────────┐                    ┌────────────────┐
 │ QUERY TYPES   │                    │    CACHING     │                    │  PROTOCOLS     │
 └──────┬────────┘                    └───────┬────────┘                    └──────┬─────────┘
        │                                     │                                    │
        ▼                                     ▼                                    ▼

 • Recursive                      • Browser Cache                   • UDP Port 53
 • Iterative                      • OS Cache                        • TCP Port 53
 • Non-Recursive                  • Resolver Cache                  • TCP for Large Responses
                                  • TTL (Time To Live)


        ┌──────────────────────────────────────┼──────────────────────────────────────┐
        │                                      │                                      │
        ▼                                      ▼                                      ▼

 ┌───────────────┐                    ┌────────────────┐                    ┌────────────────┐
 │ KUBERNETES    │                    │ TROUBLESHOOT   │                    │ REAL USE CASES │
 └──────┬────────┘                    └───────┬────────┘                    └──────┬─────────┘
        │                                     │                                    │
        ▼                                     ▼                                    ▼

 • CoreDNS                        • nslookup domain                 • Website Access
 • Service Discovery              • dig domain                      • Email Routing
 • Cluster DNS                    • host domain                     • Load Balancing
 • svc.cluster.local              • Check CoreDNS Pods              • Failover
 • Pod → Service Lookup           • Check /etc/resolv.conf          • Cloud Services
                                  • Verify DNS Records              • Microservices


                                               │
                                               ▼

                                INTERVIEW MUST-KNOW QUESTIONS

    1. What is DNS?
    2. Explain DNS resolution flow.
    3. Difference between Recursive and Iterative query?
    4. What is TTL?
    5. Difference between A and CNAME record?
    6. Why does DNS use UDP?
    7. When does DNS use TCP?
    8. How DNS works in Kubernetes?
    9. What is CoreDNS?
   10. How do you troubleshoot DNS issues in Kubernetes?
```

---

# 🎯 30-Second Interview Revision

```text
DNS = Domain Name → IP Address

Flow:
Client → Resolver → Root → TLD → Authoritative → IP

Important Records:
A, AAAA, CNAME, MX, NS, TXT

Important Queries:
Recursive, Iterative

Caching:
Browser → OS → Resolver → TTL

Kubernetes:
CoreDNS provides service discovery
my-service.default.svc.cluster.local

Debug:
nslookup
dig
kubectl exec busybox -- nslookup service-name
```

## Interview Priority (0–3 Years)

⭐⭐⭐⭐⭐ DNS Resolution Flow
⭐⭐⭐⭐⭐ A vs CNAME
⭐⭐⭐⭐⭐ Recursive vs Iterative Query
⭐⭐⭐⭐⭐ CoreDNS in Kubernetes
⭐⭐⭐⭐⭐ DNS Troubleshooting Commands
⭐⭐⭐⭐ TTL & Caching
⭐⭐⭐ MX / TXT / PTR Records
⭐⭐⭐ Authoritative vs Recursive DNS Server

This mind map covers about **90% of DNS questions asked in DevOps, Cloud, Linux, Kubernetes, and Junior SRE interviews**.
