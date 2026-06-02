# SSL/TLS Mind Map (0–3 Years DevOps / Kubernetes Interview Revision)

```text
SSL / TLS
│
├── Purpose
│   ├── Encrypt data in transit
│   ├── Ensure confidentiality
│   ├── Ensure integrity
│   └── Verify server identity
│
├── Common Use Cases
│   ├── HTTPS Websites
│   ├── Kubernetes API Server
│   ├── Ingress Controllers
│   ├── Load Balancers
│   ├── API Communication
│   ├── Database Connections
│   └── Service-to-Service Communication
│
├── Key Components
│   │
│   ├── Private Key (.key)
│   │   ├── Secret
│   │   ├── Stored on server
│   │   └── Used for decryption/signing
│   │
│   ├── Public Key
│   │   ├── Shared publicly
│   │   └── Used for encryption
│   │
│   ├── CSR (.csr)
│   │   ├── Certificate Signing Request
│   │   ├── Contains public key
│   │   └── Sent to CA
│   │
│   └── Certificate (.crt/.pem)
│       ├── Issued by CA
│       ├── Contains public key
│       └── Proves server identity
│
├── Certificate Authority (CA)
│   ├── Trusted third party
│   ├── Verifies ownership
│   ├── Signs certificate
│   └── Examples
│       ├── :contentReference[oaicite:0]{index=0}
│       ├── :contentReference[oaicite:1]{index=1}
│       └── :contentReference[oaicite:2]{index=2}
│
├── SSL Certificate Generation Flow
│   │
│   ├── Generate Private Key
│   │
│   ├── Create CSR
│   │
│   ├── Send CSR to CA
│   │
│   ├── CA validates domain
│   │
│   ├── CA issues certificate
│   │
│   └── Configure Web Server
│
├── TLS Handshake (Interview Favorite)
│   │
│   ├── Client → Hello
│   │
│   ├── Server → Certificate
│   │
│   ├── Client validates certificate
│   │
│   ├── Session key generated
│   │
│   ├── Secure channel established
│   │
│   └── Encrypted communication starts
│
├── HTTPS Request Flow
│   │
│   ├── Browser requests website
│   ├── Server sends certificate
│   ├── Browser verifies CA trust
│   ├── TLS handshake completes
│   └── Data transferred securely
│
├── Certificate Types
│   ├── Self-Signed
│   │   ├── Internal testing
│   │   └── Not trusted publicly
│   │
│   ├── CA Signed
│   │   └── Production use
│   │
│   ├── Wildcard
│   │   └── *.example.com
│   │
│   └── SAN Certificate
│       └── Multiple domains
│
├── TLS Versions
│   ├── TLS 1.0 ❌ Deprecated
│   ├── TLS 1.1 ❌ Deprecated
│   ├── TLS 1.2 ✅ Common
│   └── TLS 1.3 ✅ Recommended
│
├── SSL vs TLS
│   ├── SSL = Older protocol
│   ├── TLS = Modern replacement
│   ├── SSL insecure now
│   └── Everyone says "SSL"
│       but actually means TLS
│
├── Kubernetes Usage
│   │
│   ├── API Server Certificates
│   ├── kubelet Authentication
│   ├── etcd Communication
│   ├── Ingress HTTPS
│   ├── cert-manager
│   └── Secret Storage
│
├── Useful Commands
│   │
│   ├── Generate Key + CSR
│   │   openssl req -new -newkey rsa:2048
│   │
│   ├── View Certificate
│   │   openssl x509 -in cert.crt -text
│   │
│   ├── Verify Certificate
│   │   openssl verify cert.crt
│   │
│   └── Check HTTPS Endpoint
│       openssl s_client -connect host:443
│
├── Common Interview Questions
│   ├── What is TLS?
│   ├── Difference between SSL and TLS?
│   ├── What is CSR?
│   ├── What is a CA?
│   ├── Explain TLS Handshake.
│   ├── What is a Private Key?
│   ├── What is a Public Key?
│   ├── What is a Wildcard Certificate?
│   └── How does HTTPS work?
│
└── Quick Memory Trick
    │
    ├── KEY  → Secret
    ├── CSR  → Request
    ├── CA   → Verify & Sign
    ├── CRT  → Identity Proof
    └── TLS  → Secure Communication
```

---

## 30-Second Interview Answer

> "TLS is a security protocol used to encrypt communication between a client and server. It provides confidentiality, integrity, and authentication. The server presents a certificate issued by a trusted Certificate Authority, the client verifies it, a TLS handshake establishes a session key, and all subsequent communication is encrypted. In Kubernetes, TLS is used extensively for API server, etcd, kubelet, and Ingress security."
