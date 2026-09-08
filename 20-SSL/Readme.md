# 20 - SSL/TLS Concept

## 🎯 Objective

Understand:

* Symmetric vs asymmetric encryption
* Public/private key pairs
* SSL/TLS and HTTPS
* Certificate, CSR, and Certificate Authority (CA)
* How TLS prevents Man-in-the-Middle (MITM) attacks
* How to generate and inspect keys, CSRs, and certificates using OpenSSL
* The basic TLS flow used in Kubernetes and web applications

---

# 1. Symmetric Key Encryption

**Symmetric encryption** uses the **same secret key** for encryption and decryption.

```text
             Same Secret Key
                  │
Client ── Encrypt ────────> Server
Client <─ Decrypt ───────── Server
```

### Problem

The client and server must somehow share the secret key securely.

If an attacker obtains the key:

```text
Client ──────── Attacker ──────── Server
                 │
              Secret Key
```

the attacker can potentially decrypt the communication.

### Example

* AES
* ChaCha20

> **Memory rule:**
> **Symmetric = same key + fast encryption**

---

# 2. Asymmetric Encryption

Asymmetric cryptography uses a **key pair**:

```text
Public Key  → Can be shared
Private Key → Must remain secret
```

The two keys are mathematically related.

```text
        Key Pair
       /        \
 Public Key   Private Key
```

Common algorithms:

* RSA
* ECDSA / elliptic-curve cryptography
* Ed25519

### Basic idea

Data encrypted with a public key can generally be decrypted only with the corresponding private key.

For digital signatures:

```text
Private Key → Sign
Public Key  → Verify
```

> **Memory rule:**
> **Asymmetric = public + private key**

---

# 3. Why Do We Need TLS?

Suppose you access:

```text
https://example.com
```

We need three important security properties:

### 1. Confidentiality

Attackers should not be able to read the communication.

### 2. Integrity

Attackers should not be able to silently modify the communication.

### 3. Authentication

The client should be able to verify that it is communicating with the intended server.

```text
Client
  │
  │  Is this really example.com?
  ▼
Server
```

TLS provides these protections.

---

# 4. Why Public/Private Keys Alone Are Not Enough

Suppose a server generates:

```text
Server
 ├── Public Key
 └── Private Key
```

The server sends its public key to the client.

An attacker can intercept the connection and send **their own public key** instead.

```text
Client
   │
   │ Attacker's Public Key
   ▼
Attacker
   │
   │ Server's Public Key
   ▼
Server
```

The client has no reliable way to know whether the received public key actually belongs to the intended server.

This is the **authentication problem**.

TLS solves this using **digital certificates and trusted Certificate Authorities**.

---

# 5. TLS Certificate

A TLS certificate binds:

```text
Identity / Domain
        +
    Public Key
        +
 Certificate Authority Signature
```

For example:

```text
Certificate
 ├── Subject / SAN: example.com
 ├── Public Key
 ├── Issuer: Trusted CA
 ├── Validity period
 └── CA digital signature
```

The browser validates the certificate and its chain of trust.

> **Important:** The server does **not** send a "private certificate" to the client.
> The server keeps the **private key** secret and sends the **certificate** containing the public key and identity information.

---

# 6. Certificate Authority (CA)

A **Certificate Authority** is a trusted organization that signs certificates.

Examples include:

* DigiCert
* GlobalSign
* Let's Encrypt

The browser/OS already contains a set of **trusted CA root certificates**.

The trust relationship looks like:

```text
Trusted Root CA
      │
      ▼
Intermediate CA
      │
      ▼
Server Certificate
      │
      ▼
example.com
```

If the certificate is valid and the hostname matches, the client can establish trust.

---

# 7. CSR — Certificate Signing Request

A **CSR (Certificate Signing Request)** is a request sent to a CA asking it to issue a certificate.

A CSR contains:

* Public key
* Requested identity information
* Subject/SAN information
* A signature created using the corresponding private key

The **private key is not included in the CSR**.

```text
Server
 ├── Private Key  🔒
 │
 └── CSR
       │
       ▼
      CA
       │
       ▼
Signed Certificate
```

### Important distinction

```text
Private Key → Generated and kept by you
CSR         → Sent to CA
Certificate → Issued/signed by CA
```

> **CSR itself does not prove domain ownership.**
> The CA performs its own validation before issuing the certificate.

---

# 8. Generate Private Key + CSR Using OpenSSL

OpenSSL can generate keys, CSRs, and certificates.

Generate an RSA private key and CSR:

```bash
openssl req -new \
  -newkey rsa:2048 \
  -nodes \
  -keyout example.com.key \
  -out example.com.csr
```

This creates:

```text
example.com.key   → Private Key 🔒
example.com.csr   → Certificate Signing Request
```

### Important

Never share:

```text
example.com.key
```

The private key must remain protected.

---

# 9. Inspect the CSR

```bash
openssl req \
  -in example.com.csr \
  -noout \
  -text
```

You can inspect:

* Subject
* Public key
* Signature
* Requested extensions

---

# 10. CSR → CA → Certificate

The normal public TLS certificate workflow is:

```text
1. Generate Private Key
          ↓
2. Generate CSR
          ↓
3. Submit CSR to CA
          ↓
4. CA validates domain/organization
          ↓
5. CA signs certificate
          ↓
6. Install certificate on server
```

The server now has:

```text
Private Key 🔒
Certificate
Certificate Chain
```

The private key stays on the server.

---

# 11. How HTTPS/TLS Communication Works

A simplified TLS flow:

```text
Client
  │
  │  ClientHello
  ▼
Server
  │
  │  ServerHello
  │  Certificate
  ▼
Client
  │
  │  Validate certificate
  │
  │  Establish shared session keys
  ▼
Encrypted communication
```

Modern TLS uses **asymmetric cryptography for authentication/key establishment** and **symmetric cryptography for the actual application data** because symmetric encryption is much faster.

```text
Asymmetric
    ↓
Authentication + key establishment
    ↓
Shared session key
    ↓
Symmetric encryption
    ↓
Application data
```

> **CKA memory rule:**
> **TLS does not encrypt all application traffic using the server's RSA public key.**

---

# 12. How TLS Prevents MITM

Without certificate validation:

```text
Client ───────> Attacker ───────> Server
```

The attacker could impersonate the server.

With TLS:

```text
Client
  │
  │ Server Certificate
  ▼
Validate:
  ├── Trusted CA?
  ├── Valid certificate?
  ├── Correct hostname?
  ├── Not expired?
  └── Valid certificate chain?
          │
          ▼
       Trusted
```

If the attacker presents a certificate that is not trusted for the requested hostname, the client warns or rejects the connection.

---

# 13. Important Certificate Fields

### Common Name (CN)

Historically used for the certificate's primary name.

Example:

```text
CN=example.com
```

### SAN — Subject Alternative Name

Modern TLS hostname validation uses **SAN**.

Example:

```text
DNS:example.com
DNS:www.example.com
```

> **CKA/real-world tip:** When generating certificates today, make sure the required hostname is present in **SAN**. Do not rely only on CN.

---

# 14. Public vs Internal Certificates

## Public Domain

For a public website:

```text
example.com
      ↓
Public CA
      ↓
Browser trusts certificate
```

The certificate is publicly trusted because the CA chains to a root trusted by the client's OS/browser.

---

## Internal / Intranet

For internal applications:

```text
internal.example.local
        ↓
Internal CA
        ↓
Internal certificate
```

An organization can operate its own CA.

Clients must trust that organization's root CA.

> **Correction:** Public CAs are not the only way to use TLS. Internal domains can also use certificates issued by an organization's private/internal CA.

---

# 17. Certificate Files — Remember This

```text
.key  → Private Key 🔒
.csr  → Certificate Signing Request
.crt  → Certificate
.pem  → Container/encoding format commonly used for keys/certs
```

The file extension alone does not determine the cryptographic type; inspect the file/content when necessary.

---

# 18. SSL vs TLS

You will often hear:

```text
SSL certificate
SSL/TLS certificate
HTTPS certificate
```

Modern systems use **TLS**, not the old SSL protocols.

```text
SSL → Older, deprecated
TLS → Modern protocol
```

So technically:

```text
HTTPS = HTTP over TLS
```

---

# 19. CKA / Kubernetes Connection

In Kubernetes, TLS certificates are commonly stored in a **Secret**.

For example:

```bash
kubectl create secret tls my-tls \
  --cert=tls.crt \
  --key=tls.key
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

Ingress controllers can use this Secret to terminate HTTPS traffic.

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

> **CKA connection:**
> `kubectl create secret tls` is a command worth remembering.

---

