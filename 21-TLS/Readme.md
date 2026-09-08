# 21 — Manage TLS Certificates in Kubernetes

## Objective

By the end of this topic, you should understand:

* How Kubernetes uses **TLS certificates for authentication**
* The relationship between **client, server, CA, certificate, and private key**
* How to create a **Certificate Signing Request (CSR)**
* How to submit, approve, and retrieve a Kubernetes CSR
* How certificates are used by **kubectl, kube-apiserver, kubelet, and other control-plane components**
* How to use the issued certificate in a **kubeconfig**

---

# 1. Kubernetes Certificate Architecture

Kubernetes uses certificates to establish **authenticated and encrypted TLS communication** between components.

### Common communication paths

```text
kubectl
   ↓ TLS
kube-apiserver
   ↓ TLS
kubelet
```

Other control-plane components also communicate with the API server:

```text
Scheduler ────────┐
Controller Manager ──→ kube-apiserver
kubelet ──────────┘
```

The **kube-apiserver** acts as a server when receiving requests and may act as a client when connecting to other components such as:

```text
kube-apiserver → etcd
kube-apiserver → kubelet
```

### CKA Mental Model

> **Every TLS connection has a client, a server, and certificates used to establish trust.**

---

# 2. Certificate vs Private Key

### Certificate

Usually stored as:

```text
.crt
.pem
```

A certificate contains information such as:

* Identity
* Public key
* Issuer
* Validity period
* CA signature

### Private Key

Usually stored as:

```text
.key
```

The private key must remain secret.

```text
Certificate → Public information
Private Key → Secret
```

> **Never treat `.pem` as automatically meaning "certificate".** PEM is an encoding/container format and can contain certificates, private keys, CSRs, etc.

---

# 3. Certificate Authority (CA)

A **Certificate Authority (CA)** signs certificates and establishes trust.

Simplified flow:

```text
User / Component
      │
      │ CSR
      ▼
     CA
      │
      │ Signed Certificate
      ▼
User / Component
```

Kubernetes clusters commonly have an **internal CA** rather than using a public CA such as DigiCert.

### Important

The CA has its own:

```text
CA Private Key
CA Certificate
```

The CA uses its **private key** to sign certificates.

Clients use the **CA certificate** to verify the signature and establish trust.

---

# 4. Types of Certificates in Kubernetes

Common certificate roles include:

1. **Client certificate** — authenticates a client to the API server
2. **Server certificate** — authenticates a server
3. **CA certificate** — establishes the trust chain

For the CKA, the **client certificate + CSR workflow** is especially important.

---

# 5. Create a Certificate for a New Kubernetes User

### Scenario

A new user, **Adam**, needs access to the Kubernetes cluster.

We will:

```text
Generate private key
        ↓
Create CSR
        ↓
Submit CSR to Kubernetes
        ↓
Approve CSR
        ↓
Retrieve signed certificate
        ↓
Configure kubeconfig
```

---

# 6. Step 1 — Generate Private Key

Generate Adam's private key:

```bash
openssl genrsa -out adam.key 2048
```

Verify:

```bash
ls -l adam.key
```

The private key **stays with Adam** and should not be shared publicly.

---

# 7. Step 2 — Generate the CSR

Create a Certificate Signing Request:

```bash
openssl req -new \
  -key adam.key \
  -out adam.csr \
  -subj "/CN=adam"
```

Here:

```text
adam.key → Private key
adam.csr → Certificate Signing Request
CN=adam  → Kubernetes username
```

The CSR contains the public key derived from the private key and identity information requested for the certificate.

### Inspect the CSR

```bash
openssl req -in adam.csr -noout -text
```

---

# 8. Step 3 — Base64-Encode the CSR

Kubernetes CSR objects expect the CSR data in base64.

```bash
cat adam.csr | base64 | tr -d '\n'
```

Meaning:

```text
base64 → Encodes the CSR
tr -d '\n' → Removes line breaks
```

> **Base64 is encoding, not encryption.**

Copy the resulting single-line value.

---

# 9. Step 4 — Create Kubernetes CSR

Create `adam-csr.yaml`:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: adam
spec:
  request: <BASE64_ENCODED_CSR>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400
  usages:
    - client auth
```

Apply it:

```bash
kubectl apply -f adam-csr.yaml
```

Check the CSR:

```bash
kubectl get csr
```

Initially:

```text
NAME    AGE   SIGNERNAME                            REQUESTOR   CONDITION
adam    5s    kubernetes.io/kube-apiserver-client  ...         Pending
```

---

# 10. Step 5 — Approve the CSR

An authorized administrator can approve it:

```bash
kubectl certificate approve adam
```

Check:

```bash
kubectl get csr
```

You should see:

```text
Approved
```

Detailed information:

```bash
kubectl describe csr adam
```

### CKA Shortcut

```bash
kubectl certificate approve <csr-name>
```

---

# 11. Step 6 — Retrieve the Signed Certificate

Retrieve the issued certificate:

```bash
kubectl get csr adam \
  -o jsonpath='{.status.certificate}'
```

Decode it:

```bash
kubectl get csr adam \
  -o jsonpath='{.status.certificate}' \
  | base64 --decode > adam.crt
```

Now you have:

```text
adam.key   → Private key
adam.csr   → Certificate Signing Request
adam.crt   → Signed client certificate
```

Verify the certificate:

```bash
openssl x509 -in adam.crt -noout -text
```

Check its subject:

```bash
openssl x509 -in adam.crt -noout -subject
```

You should see the identity associated with:

```text
CN=adam
```

---

# 12. Certificate + Permissions

A certificate authenticates the user, but **does not determine what the user is allowed to do**.

Kubernetes separates:

```text
Authentication → Who are you?
Authorization  → What can you do?
```

For example:

```text
Certificate
   ↓
Authenticates Adam
   ↓
RBAC
   ↓
Determines Adam's permissions
```

To grant permissions, use:

* Role / ClusterRole
* RoleBinding / ClusterRoleBinding

---

# 13. Use the Certificate in kubeconfig

Configure credentials:

```bash
kubectl config set-credentials adam \
  --client-certificate=adam.crt \
  --client-key=adam.key
```

Configure a context:

```bash
kubectl config set-context adam \
  --cluster=<cluster-name> \
  --user=adam
```

Switch to the user:

```bash
kubectl config use-context adam
```

Test:

```bash
kubectl auth can-i get pods
```

The result depends on the RBAC permissions assigned to Adam.

---

# 14. Complete CKA Workflow

Memorize this workflow:

```text
openssl genrsa
      ↓
adam.key
      ↓
openssl req -new
      ↓
adam.csr
      ↓
base64 encode
      ↓
CertificateSigningRequest YAML
      ↓
kubectl apply
      ↓
Pending
      ↓
kubectl certificate approve
      ↓
Approved
      ↓
Extract .status.certificate
      ↓
base64 --decode
      ↓
adam.crt
      ↓
kubeconfig
      ↓
RBAC permissions
```

---

# 15. Important CKA Commands

### Generate private key

```bash
openssl genrsa -out adam.key 2048
```

### Generate CSR

```bash
openssl req -new -key adam.key -out adam.csr -subj "/CN=adam"
```

### Encode CSR

```bash
cat adam.csr | base64 | tr -d '\n'
```

### Create CSR

```bash
kubectl apply -f adam-csr.yaml
```

### Check CSR

```bash
kubectl get csr
```

### Approve

```bash
kubectl certificate approve adam
```

### Describe

```bash
kubectl describe csr adam
```

### Extract certificate

```bash
kubectl get csr adam \
  -o jsonpath='{.status.certificate}' \
  | base64 --decode > adam.crt
```

### Inspect certificate

```bash
openssl x509 -in adam.crt -noout -text
```

---

# 16. Troubleshooting

### CSR remains Pending

Check:

```bash
kubectl get csr
kubectl describe csr adam
```

Possible reason:

```text
CSR has not been approved
```

Approve:

```bash
kubectl certificate approve adam
```

---

### Certificate is not generated

Check:

```bash
kubectl describe csr adam
```

Make sure the CSR has:

```text
Approved
```

Then:

```bash
kubectl get csr adam -o yaml
```

Look for:

```yaml
status:
  certificate:
```

---

### User can authenticate but cannot perform an action

Authentication is working.

Check authorization:

```bash
kubectl auth can-i get pods --as=adam
```

Then inspect the relevant:

```text
Role
RoleBinding
ClusterRole
ClusterRoleBinding
```

---

# 17. CKA Cheat Sheet

| Concept                       | Remember                    |
| ----------------------------- | --------------------------- |
| `.key`                        | Private key                 |
| `.crt`                        | Certificate                 |
| `.csr`                        | Certificate Signing Request |
| CA                            | Signs certificates          |
| Base64                        | Encoding, not encryption    |
| Certificate                   | Authenticates identity      |
| RBAC                          | Controls permissions        |
| `kubectl get csr`             | View CSRs                   |
| `kubectl certificate approve` | Approve CSR                 |
| `.status.certificate`         | Issued certificate          |
| `CN=adam`                     | Kubernetes username         |
| `client auth`                 | Client authentication usage |

### One-line memory rule

> **Key → CSR → Kubernetes CSR → Approve → Certificate → kubeconfig → RBAC**

### Most Important CKA Point

**Certificate = Authentication.
RBAC = Authorization.**
