# 18. Inspect the Certificate

### View complete certificate details

```bash
openssl x509 \
  -in example.com.crt \
  -noout \
  -text
```

### Check subject

```bash
openssl x509 \
  -in example.com.crt \
  -noout \
  -subject
```

### Check issuer

```bash
openssl x509 \
  -in example.com.crt \
  -noout \
  -issuer
```

For a self-signed certificate:

```text
Subject = Issuer
```

### Check validity period

```bash
openssl x509 \
  -in example.com.crt \
  -noout \
  -dates
```

Example:

```text
notBefore=Sep  7 10:00:00 2026 GMT
notAfter=Sep  7 10:00:00 2027 GMT
```

### Check SAN

```bash
openssl x509 \
  -in example.com.crt \
  -noout \
  -ext subjectAltName
```

Expected:

```text
X509v3 Subject Alternative Name:
    DNS:example.com, DNS:www.example.com
```

> **CKA/Real-world tip:** Always verify the required hostname under **SAN**.

---

# 19. Verify Certificate and Private Key Match

The certificate must contain the public key corresponding to the private key.

### Extract public key from certificate

```bash
openssl x509 \
  -in example.com.crt \
  -pubkey \
  -noout > cert.pub
```

### Extract public key from private key

```bash
openssl pkey \
  -in example.com.key \
  -pubout > key.pub
```

### Compare

```bash
diff cert.pub key.pub
```

If there is **no output**, the keys match.

```text
Private Key
     │
     └── Public Key
             │
             ▼
      Certificate Public Key

          MATCH ✅
```

---

# 20. Verify Certificate Chain / Trust

For a self-signed certificate:

```bash
openssl verify example.com.crt
```

You will normally see:

```text
error 18 at 0 depth lookup: self-signed certificate
```

This is expected because the certificate is **not signed by a CA trusted by OpenSSL**.

You can explicitly trust the certificate for the test:

```bash
openssl verify -CAfile example.com.crt example.com.crt
```

Expected:

```text
example.com.crt: OK
```

### Important distinction

```text
Self-Signed Certificate
        ↓
Not automatically trusted

CA-Signed Certificate
        ↓
Can be trusted when CA chain is trusted
```

---

# 21. Verify Certificate Against a Real TLS Server

For a server using HTTPS:

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com
```

Useful information includes:

* Server certificate
* Certificate chain
* TLS version
* Cipher
* Verification result

Look for:

```text
Verify return code: 0 (ok)
```

### Check only certificate dates

```bash
echo | openssl s_client \
  -connect example.com:443 \
  -servername example.com 2>/dev/null |
openssl x509 -noout -dates
```

### Inspect the remote certificate

```bash
echo | openssl s_client \
  -connect example.com:443 \
  -servername example.com 2>/dev/null |
openssl x509 -noout -subject -issuer -dates
```

> `-servername` sends the **SNI hostname**, which is important when a server hosts multiple HTTPS domains.

---

# 22. Certificate Inspection Cheat Sheet

```text
CSR
 │
 ├── Inspect
 │   openssl req -in file.csr -noout -text
 │
 └── Verify
     openssl req -in file.csr -noout -verify
```

```text
Certificate
 │
 ├── Details
 │   openssl x509 -in cert.crt -noout -text
 │
 ├── Subject
 │   openssl x509 -in cert.crt -noout -subject
 │
 ├── Issuer
 │   openssl x509 -in cert.crt -noout -issuer
 │
 ├── Validity
 │   openssl x509 -in cert.crt -noout -dates
 │
 └── SAN
     openssl x509 -in cert.crt -noout -ext subjectAltName
```

```text
Private Key
 │
 └── Verify
     openssl pkey -in key.pem -check -noout
```

```text
Certificate + Private Key
          │
          ▼
       Compare
          │
          ▼
       Public Keys
          │
          ├── Same → MATCH ✅
          └── Different → WRONG KEY ❌
```

---