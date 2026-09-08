## 15. Hands-On — Generate a SAN-Based Private Key + CSR

For modern TLS certificates, include the hostname in the **Subject Alternative Name (SAN)**.

### Step 1 — Create a working directory

```bash
mkdir tls-lab
cd tls-lab
```

### Step 2 — Create an OpenSSL configuration

Create `san.cnf`:

```ini
[req]
distinguished_name = req_distinguished_name
req_extensions = req_ext
prompt = no

[req_distinguished_name]
C = IN
ST = Bihar
L = Patna
O = Example Company
CN = example.com

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
```

Here:

```text
CN  → example.com
SAN → example.com
      www.example.com
```

### Step 3 — Generate Private Key + CSR

```bash
openssl req -new \
  -newkey rsa:2048 \
  -nodes \
  -keyout example.com.key \
  -out example.com.csr \
  -config san.cnf
```

This creates:

```text
example.com.key → Private Key 🔒
example.com.csr → CSR
```

### Step 4 — Verify the CSR

```bash
openssl req \
  -in example.com.csr \
  -noout \
  -text
```

Look for:

```text
X509v3 Subject Alternative Name:
    DNS:example.com
    DNS:www.example.com
```

### Verify CSR subject
```
openssl req \
  -in example.com.csr \
  -noout \
  -subject
```
Expected:

```
subject=C=IN, ST=Bihar, L=Patna, O=Example Company, CN=example.com
```

### Verify the CSR signature
```
openssl req \
  -in example.com.csr \
  -noout \
  -verify
```

Expected:

Certificate request self-signature verify OK

### Step 5 — Verify the private key

```bash
openssl pkey \
  -in example.com.key \
  -check \
  -noout
```

Expected:

```text
Key is valid
```

> Memory rule:
- openssl req -text → inspect CSR
- openssl req -verify → verify CSR signature
- openssl pkey -check → verify private key

> **Real-world tip:** 
The hostname clients connect to should be present in the certificate's **SAN**. Do not rely only on the CN.

---

# 16. Hands-On — Create a SAN-Based Self-Signed Certificate

For a local lab, you can create a self-signed certificate using the same SAN configuration.

```bash
openssl req -x509 \
  -new \
  -key example.com.key \
  -out example.com.crt \
  -days 365 \
  -config san.cnf \
  -extensions req_ext
```

Now you have:

```text
example.com.key → Private Key 🔒
example.com.csr → CSR
example.com.crt → Self-signed Certificate
```

Verify the certificate:

```bash
openssl x509 \
  -in example.com.crt \
  -noout \
  -text
```

Check SAN specifically:

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

### Verify certificate and private key match

Check the public key from the certificate:

```bash
openssl x509 -in example.com.crt -pubkey -noout > cert.pub
```

Check the public key derived from the private key:

```bash
openssl pkey -in example.com.key -pubout > key.pub
```

Compare:

```bash
diff cert.pub key.pub
```

No output means they match.

> **Important:** A self-signed certificate is useful for labs/testing, but browsers and other clients will not normally trust it unless you explicitly trust its CA/certificate.

---

# 🔑 SAN Quick Memory

```text
Certificate
     │
     ├── CN  → Legacy/common name
     │
     └── SAN → Modern hostname validation
```

Example:

```text
SAN:
  DNS:example.com
  DNS:www.example.com
```

If the client connects to:

```text
https://www.example.com
```

`www.example.com` must be covered by the certificate's SAN.
---