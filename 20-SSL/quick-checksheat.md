# 23. Certificate Files — Remember This

```text
.key  → Private Key 🔒
.csr  → Certificate Signing Request
.crt  → Certificate
.pem  → Common encoding/container format
```

> File extensions are conventions; the actual content determines what the file contains.

---

# 24. Kubernetes Connection

TLS certificates are commonly stored in a Kubernetes **Secret**:

```bash
kubectl create secret tls my-tls \
  --cert=example.com.crt \
  --key=example.com.key
```

Verify:

```bash
kubectl get secret my-tls
```

Typical structure:

```text
Secret
 ├── tls.crt
 └── tls.key
```

Ingress controllers can use this Secret for TLS termination:

```text
Client
   │
   │ HTTPS
   ▼
Ingress
   │
   │ TLS termination
   ▼
Service
   │
   ▼
Pod
```

> **CKA command to remember:**

```bash
kubectl create secret tls <name> \
  --cert=<certificate> \
  --key=<private-key>
```

---

# 25. SSL vs TLS

Modern systems use **TLS**, not the old SSL protocols.

```text
SSL → Old / deprecated
TLS → Modern protocol
```

Therefore:

```text
HTTPS = HTTP over TLS
```

---

# 26. Final Cheat Sheet

```text
Symmetric
    ↓
Same secret key
    ↓
Fast bulk-data encryption
```

```text
Asymmetric
    ↓
Public + Private key
    ↓
Authentication / signatures / key establishment
```

```text
CSR
    ↓
Public key + identity/request information
    ↓
CA
    ↓
Certificate
```

```text
Certificate
    ↓
Identity + Public Key + CA Signature
    ↓
SAN → Modern hostname validation
```

```text
TLS
    ↓
Authenticate
    ↓
Establish session keys
    ↓
Encrypt application traffic
```

```text
Kubernetes
    ↓
TLS Secret
    ├── tls.crt
    └── tls.key
```

### 🔑 Must Remember

* **Private key → never share**
* **CSR → certificate request**
* **Certificate → identity + public key**
* **SAN → hostname validation**
* **CA → signs certificate**
* **Self-signed → useful for labs, not publicly trusted**
* **`openssl req` → CSR operations**
* **`openssl x509` → certificate operations**
* **`openssl pkey` → private/public key operations**
* **`openssl s_client` → inspect a live TLS server**
* **HTTPS → HTTP over TLS**
* **Kubernetes TLS → Secret**
