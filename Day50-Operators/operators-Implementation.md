# Kubernetes Operators — Hands-On Lab

## Scenario: Secure Internal API with Automated TLS

> **Role:** You are a DevOps Engineer at a fintech startup. The backend team has built a new
> payments API (`payments-api`) that must be served over HTTPS — even inside the cluster —
> to satisfy your company's security policy. You have been asked to automate TLS certificate
> issuance and renewal so the team never has to touch a certificate manually again.
>
> **You will use the Cert-Manager Operator** to deploy a self-signed CA, issue a TLS
> certificate automatically, mount it into the payments-api deployment, and simulate what
> happens when the Operator's reconciliation loop kicks in.

---

## Prerequisites

| Requirement      | Version | Check                               |
| ---------------- | ------- | ----------------------------------- |
| kubectl          | ≥ 1.27  | `kubectl version --client`          |
| kind or minikube | latest  | `kind version` / `minikube version` |
| helm             | ≥ 3.12  | `helm version`                      |
| curl / openssl   | any     | `openssl version`                   |

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────┐
│  Kubernetes Cluster                                     │
│                                                         │
│  ┌──────────────────────┐    watches    ┌────────────┐  │
│  │  cert-manager        │◄───────────── │ Certificate│  │
│  │  Operator (Pod)      │               │ CR         │  │
│  └────────┬─────────────┘               └────────────┘  │
│           │ issues & renews                             │
│           ▼                                             │
│  ┌────────────────┐      mounts       ┌─────────────┐   │
│  │ TLS Secret     │◄──────────────────│ payments-api│   │
│  │ (tls.crt/key)  │                   │ Deployment  │   │
│  └────────────────┘                   └─────────────┘   │
│                                                         │
│  ClusterIssuer (self-signed CA) ──► signs certificates  │
└─────────────────────────────────────────────────────────┘
```

---

## Lab Structure

| Phase | Task                 | Goal                             |
| ----- | -------------------- | -------------------------------- |
| 1     | Cluster setup        | Working local cluster            |
| 2     | Install cert-manager | Operator running                 |
| 3     | Create ClusterIssuer | Self-signed CA ready             |
| 4     | Issue Certificate    | TLS Secret created automatically |
| 5     | Deploy payments-api  | App serving with TLS mounted     |
| 6     | Break & Watch        | See reconciliation in action     |
| 7     | Cleanup              |                                  |

---

## Phase 1 — Create a Local Cluster

```bash
# Using kind (recommended)
kind create cluster --name fintech-lab

# OR using minikube
minikube start --profile fintech-lab

# Verify
kubectl cluster-info
kubectl get nodes
```

Expected output:

```
NAME                        STATUS   ROLES           AGE
fintech-lab-control-plane   Ready    control-plane   30s
```

---

## Phase 2 — Install cert-manager Operator

We install cert-manager using Helm (the most production-common method).

```bash
# Add the Jetstack Helm repo
helm repo add jetstack https://charts.jetstack.io
helm repo update

# Install cert-manager with CRDs included
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true

# Wait for all 3 Operator pods to be Running
kubectl get pods -n cert-manager --watch
```

Expected output (wait until all 3 are Running):

```
NAME                                       READY   STATUS    RESTARTS
cert-manager-xxxxxxxxx-xxxxx              1/1     Running   0
cert-manager-cainjector-xxxxxxxxx-xxxxx   1/1     Running   0
cert-manager-webhook-xxxxxxxxx-xxxxx      1/1     Running   0
```

**What just happened?** The Helm chart deployed the cert-manager Operator (3 pods) and
registered the CRDs — `Certificate`, `Issuer`, `ClusterIssuer`, and others — into your
cluster's API. You can verify:

```bash
kubectl get crds | grep cert-manager.io
```

---

## Phase 3 — Create a ClusterIssuer (Self-Signed CA)

A `ClusterIssuer` is a Custom Resource that tells cert-manager _how_ to sign certificates.
We will create a two-tier setup — a root CA that signs a CA cert, which then issues
leaf certificates for our app.

Create the file `issuer.yaml`:

```yaml
# issuer.yaml
---
# Step 1: A bootstrap self-signed issuer
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: selfsigned-bootstrap
spec:
  selfSigned: {}

---
# Step 2: A root CA certificate (signed by the bootstrap issuer)
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: fintech-root-ca
  namespace: cert-manager
spec:
  isCA: true
  commonName: fintech-root-ca
  secretName: fintech-root-ca-secret
  privateKey:
    algorithm: ECDSA
    size: 256
  issuerRef:
    name: selfsigned-bootstrap
    kind: ClusterIssuer

---
# Step 3: The real ClusterIssuer backed by our root CA
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: fintech-ca-issuer
spec:
  ca:
    secretName: fintech-root-ca-secret
```

Apply and verify:

```bash
kubectl apply -f issuer.yaml

# Check ClusterIssuers
kubectl get clusterissuer

# Check the root CA certificate
kubectl get certificate -n cert-manager

# It should show READY = True within ~10 seconds
```

Expected:

```
NAME                  READY   SECRET                 AGE
fintech-root-ca       True    fintech-root-ca-secret 10s
```

---

## Phase 4 — Issue a TLS Certificate for payments-api

Now we create a `Certificate` CR — this is the _desired state_ we declare. The Operator
will watch it and fulfill it automatically.

Create `cert.yaml`:

```yaml
# cert.yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: payments-api-tls
  namespace: default
spec:
  secretName: payments-api-tls-secret # cert-manager will create this Secret
  duration: 2160h # 90 days
  renewBefore: 360h # Renew 15 days before expiry
  commonName: payments-api.default.svc.cluster.local
  dnsNames:
    - payments-api
    - payments-api.default
    - payments-api.default.svc
    - payments-api.default.svc.cluster.local
  issuerRef:
    name: fintech-ca-issuer
    kind: ClusterIssuer
```

Apply and watch the Operator work:

```bash
kubectl apply -f cert.yaml

# Watch the certificate status in real time
kubectl get certificate payments-api-tls --watch
```

Within seconds the Operator's reconciliation loop fires:

```
NAME                READY   SECRET                     AGE
payments-api-tls    False   payments-api-tls-secret    2s
payments-api-tls    True    payments-api-tls-secret    6s
```

**False → True** is the reconciliation loop in action.

Now verify the Secret was automatically created:

```bash
kubectl get secret payments-api-tls-secret

# Inspect what's inside
kubectl describe secret payments-api-tls-secret
```

Expected:

```
Type:  kubernetes.io/tls

Data
====
ca.crt:   <N bytes>
tls.crt:  <N bytes>
tls.key:  <N bytes>
```

Decode and inspect the certificate:

```bash
kubectl get secret payments-api-tls-secret \
  -o jsonpath='{.data.tls\.crt}' | base64 -d | openssl x509 -text -noout \
  | grep -E "(Subject:|DNS:|Not Before|Not After)"
```

You should see your DNS SANs and the 90-day validity window — exactly what you declared
in the CR.

---

## Phase 5 — Deploy the payments-api Application

Create `payments-api.yaml`:

```yaml
# payments-api.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payments-api
  namespace: default
  labels:
    app: payments-api
spec:
  replicas: 2
  selector:
    matchLabels:
      app: payments-api
  template:
    metadata:
      labels:
        app: payments-api
    spec:
      containers:
        - name: payments-api
          image: nginx:alpine # Using nginx to simulate an HTTPS backend
          ports:
            - containerPort: 443
          volumeMounts:
            - name: tls-certs
              mountPath: /etc/nginx/ssl # TLS cert mounted here
              readOnly: true
          command: ["/bin/sh", "-c"]
          args:
            - |
              cat > /etc/nginx/conf.d/default.conf << 'EOF'
              server {
                listen 443 ssl;
                ssl_certificate     /etc/nginx/ssl/tls.crt;
                ssl_certificate_key /etc/nginx/ssl/tls.key;
                location /health { return 200 '{"status":"ok","tls":"enabled"}'; }
                location /       { return 200 '{"service":"payments-api","version":"1.0"}'; }
              }
              EOF
              nginx -g 'daemon off;'
      volumes:
        - name: tls-certs
          secret:
            secretName: payments-api-tls-secret # The Secret cert-manager created

---
apiVersion: v1
kind: Service
metadata:
  name: payments-api
  namespace: default
spec:
  selector:
    app: payments-api
  ports:
    - name: https
      port: 443
      targetPort: 443
```

Deploy:

```bash
kubectl apply -f payments-api.yaml

# Wait for pods to be Running
kubectl get pods -l app=payments-api --watch
```

Verify TLS is actually working from inside the cluster:

```bash
# Port-forward to test locally
kubectl port-forward svc/payments-api 8443:443 &

# Hit the health endpoint (skip CA verification since it's self-signed)
curl -k https://localhost:8443/health
# Expected: {"status":"ok","tls":"enabled"}

# Verify with the actual CA cert
kubectl get secret payments-api-tls-secret \
  -o jsonpath='{.data.ca\.crt}' | base64 -d > /tmp/ca.crt

curl --cacert /tmp/ca.crt \
     --resolve payments-api.default.svc.cluster.local:8443:127.0.0.1 \
     https://payments-api.default.svc.cluster.local:8443/health
# Expected: {"status":"ok","tls":"enabled"}
```

---

## Phase 6 — Break It and Watch the Reconciliation Loop

This is the most important part of the demo. It proves the Operator is actively
maintaining the desired state — not just a one-time setup tool.

### Test 1: Delete the Secret manually

The desired state says the Secret must exist. Let's destroy it and watch the Operator
recover it automatically.

```bash
# Open a watch in terminal 1
kubectl get secret payments-api-tls-secret --watch &

# In the same terminal: delete the secret
kubectl delete secret payments-api-tls-secret
```

Watch the output:

```
NAME                        TYPE                DATA   AGE
payments-api-tls-secret     kubernetes.io/tls   3      2m30s
payments-api-tls-secret     kubernetes.io/tls   0      0s       # Deleted
payments-api-tls-secret     kubernetes.io/tls   3      3s       # Re-created by Operator!
```

The cert-manager Operator detected that the Secret was gone (desired state != actual state)
and immediately re-issued the certificate. You did nothing.

### Test 2: Watch Operator events

```bash
# See the events the Operator emitted
kubectl describe certificate payments-api-tls

# Look at the Events section at the bottom:
# Normal  Issuing    cert-manager  Issuing certificate as Secret does not exist
# Normal  Generated  cert-manager  Stored new private key in temporary Secret
# Normal  Requested  cert-manager  Created new CertificateRequest resource
# Normal  Issuing    cert-manager  The certificate has been successfully issued
```

This is the reconciliation loop being logged in real time.

### Test 3: Force immediate renewal

In production, you might need to rotate a certificate urgently (after a suspected key
compromise). cert-manager provides a command for this.

```bash
# Install the cert-manager kubectl plugin (cmctl)
# macOS:
brew install cmctl

# Linux:
curl -fsSL -o cmctl https://github.com/cert-manager/cmctl/releases/latest/download/cmctl_linux_amd64
chmod +x cmctl && sudo mv cmctl /usr/local/bin/

# Force renewal
cmctl renew payments-api-tls -n default

# Watch the certificate cycle through False -> True again
kubectl get certificate payments-api-tls --watch
```

---

## Phase 7 — Cleanup

```bash
# Stop port-forward
kill %1

# Delete application resources
kubectl delete -f payments-api.yaml
kubectl delete -f cert.yaml
kubectl delete -f issuer.yaml

# Uninstall cert-manager
helm uninstall cert-manager -n cert-manager
kubectl delete namespace cert-manager

# Delete the cluster
kind delete cluster --name fintech-lab
# OR
minikube delete --profile fintech-lab
```

---

## What You Can Showcase

When presenting this lab, walk through these talking points in order:

1. **The problem** — manual TLS management is error-prone: expired certs cause outages,
   key rotation is painful, humans forget renewal schedules.

2. **The Operator pattern** — you declared _what you want_ (a valid TLS certificate for
   payments-api, renewed automatically), not _how to get it_. The Operator handles the how.

3. **The reconciliation loop** — demonstrate Phase 6 live. Delete the Secret in front of
   the interviewer and let the audience watch it come back within seconds. This is the
   single most memorable moment in the demo.

4. **Day-2 operations** — point out `renewBefore: 360h`. In a manual world this requires
   a calendar reminder, a human action, and a restart. With an Operator: nothing. It just happens.

5. **CRD as an API extension** — run `kubectl get crds | grep cert-manager.io` and
   show that cert-manager added its own resource types to Kubernetes. Your cluster now
   _understands_ the concept of a Certificate natively.

---

## Common Interview Questions & Answers

**Q: What is the difference between a Deployment and an Operator?**  
A: A Deployment manages stateless pods. An Operator extends Kubernetes with domain knowledge
for a specific application — it understands concepts like "certificate renewal" or "database
failover" that a generic Deployment has no awareness of.

**Q: What is a CRD vs a CR?**  
A: A CRD is the _schema_ (like a class definition). A CR is an _instance_ of that schema
(like an object). The `Certificate` CRD defines what fields a certificate resource can have;
`payments-api-tls` is a specific CR — one instance of that type.

**Q: What is the reconciliation loop?**  
A: It is the core control loop of an Operator. It continuously compares the desired state
(what the CR says) against the actual state (what exists in the cluster). When they differ,
it takes action to close the gap. You demonstrated this in Phase 6 when you deleted the Secret.

**Q: Why not just use a CronJob for certificate renewal?**  
A: A CronJob runs on a schedule regardless of state. The Operator is _event-driven_ — it
reacts immediately when a certificate is about to expire, when a secret is deleted, or when
a new Certificate CR appears. It also has full access to the Kubernetes API and can handle
complex multi-step workflows that a shell script in a CronJob cannot.

---

_Lab designed for CKA preparation — Operators_
