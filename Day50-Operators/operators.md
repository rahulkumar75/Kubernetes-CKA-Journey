# Kubernetes Operators
---

## What is an Operator?

A Kubernetes Operator is a software extension that uses custom resources to manage applications and their components. It encodes operational knowledge — the kind a human expert would use — and automates it inside Kubernetes.

---

## Manual vs. Automated Approach

### Manual (Traditional) Approach

In a native Kubernetes setup without any automation tooling, a DevOps Engineer:

- Deploys the application as a Pod or Deployment using a YAML file, Helm chart, or Kustomize.
- Handles all configuration manually.
- Is responsible for keeping the application **up-to-date** — applying image updates, vulnerability patches, and code changes.
- Must **scale** the application manually when needed (unless a separate tool like HPA, VPA, or KEDA is configured).
- Must **perform rollbacks** manually whenever an issue arises.

> **Note:** CI/CD pipelines can automate parts of this, but there is still significant operational overhead.

---

### Automated Approach — Using an Operator

With an Operator, the DevOps Engineer:

1. **Deploys the Operator** (not the application directly).
2. **Defines the desired state** in the Operator's configuration (Custom Resource).
3. The Operator then autonomously handles:
   - Configuration changes
   - Deployment changes
   - Scaling, updating, and rollbacks

#### Benefits

- Simple installation — typically a few commands via OLM or Helm.
- Easier to manage ongoing operations.
- Operators can be scoped to a single concern, keeping responsibilities isolated.
- Handles Day-2 operations (backup, restore, updates, scaling) without requiring separate tooling.

---

## How Operators Work

1. A DevOps Engineer installs the **Operator**.
2. The Operator introduces **Custom Resource Definitions (CRDs)** — schema definitions for new resource types.
3. The engineer creates **Custom Resources (CRs)** — instances of those CRDs that declare the desired state.
4. The Operator continuously **watches the CR** for any changes.
5. When a change is detected, the Operator reconciles the actual cluster state with the desired state by interacting with the **Kubernetes API**.

This continuous loop is called the **Reconciliation Loop**.

> **Core principle:** The Operator ensures that `desired state == actual state` at all times.

---

### How is it Different from GitOps?

| | GitOps | Operator |
|---|---|---|
| **Source of truth** | Git repository | Custom Resource (CR) |
| **Scope** | Deployment configs, IaC, workloads | Application-specific lifecycle logic |
| **Trigger** | Git push / pull request | Kubernetes events / CR changes |
| **Focus** | Environment configuration | Application operations |

---

## Example: Cert-Manager Operator

Cert-Manager is a popular Operator that automates TLS certificate management.

**Workflow:**

1. DevOps Engineer deploys the **Cert-Manager Operator**.
2. Cert-Manager installs its CRDs — including `Issuer` and `Certificate` types.
3. The engineer creates two CRs:
   - `Issuer` — defines *who* issues the certificate (e.g., Let's Encrypt).
   - `Certificate` — defines the actual certificate resource.
4. The Operator watches for these CRs and automatically:
   - Issues the certificate.
   - Renews the certificate before expiry.
   - Injects the certificate as a Kubernetes Secret.

**Desired state maintained by the Operator:**
- Certificate is **approved** and valid.
- Certificate is **not expired**.
- Secret is **injected** into the Kubernetes API.

---

## Key Components of an Operator

| Component | Description |
|---|---|
| **CRD (Custom Resource Definition)** | Schema that defines a new resource type in Kubernetes |
| **CR (Custom Resource)** | An instance of a CRD; holds the desired state |
| **Controller** | The core logic that watches CRs and reconciles state |
| **Reconciliation Loop** | The continuous loop that compares desired vs. actual state |

---

## Why Use Operators?

- **Automate complex tasks** that are otherwise manual.
- Handle **Day-2 Operations** natively:
  - Backup and Restore
  - Rolling Updates
  - Auto-scaling
  - Self-healing
- Replace the need for separate CI/CD pipelines, monitoring hooks, or scaling tools for specific applications.

---

## Installing Operators

There are three common ways to install an Operator:

### 1. OLM (Operator Lifecycle Manager)
OLM manages the full lifecycle of Operators — install, upgrade, and removal. It connects to a catalog of publicly available Operators.

```bash
# Install OLM first
curl -sL https://github.com/operator-framework/operator-lifecycle-manager/releases/latest/download/install.sh | bash -s <version>

# Then install an operator from the catalog
kubectl create -f <operator-subscription.yaml>
```

### 2. Helm Chart
Many Operators (like Cert-Manager) publish official Helm charts for installation.

```bash
helm repo add jetstack https://charts.jetstack.io
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace
```

### 3. Raw Manifests (YAML)
Apply the Operator's manifests directly.

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```

---

## Operator Hub

[OperatorHub.io](https://operatorhub.io) is the public catalog of community and vendor Operators. Notable examples include:

- **Cert-Manager** — TLS certificate lifecycle management
- **Prometheus Operator** — Kubernetes-native monitoring
- **Argo CD Operator** — GitOps continuous delivery

> **Tip:** OperatorHub provides installation steps but not usage guides. Always refer to the Operator's own documentation for configuration.

---

## Writing Your Own Operator

You can build a custom Operator using frameworks such as:

- **[Operator SDK](https://sdk.operatorframework.io/)** (Go, Ansible, Helm)
- **[Kubebuilder](https://book.kubebuilder.io/)** (Go)
- **[Kopf](https://kopf.readthedocs.io/)** (Python)

The general pattern is:

1. Define your CRD schema.
2. Write a Controller that watches for CR events.
3. Implement reconciliation logic to move actual state toward desired state.
4. Package and distribute via OLM or Helm.

---

## Demo: Installing Cert-Manager via OLM

### Step 1 — Verify OLM pods are running

```bash
kubectl get pods -n olm
kubectl get pods -n operators
```

### Step 2 — Confirm Cert-Manager pods are managed by the Operator

```bash
kubectl get pods -n cert-manager
```

### Step 3 — Create an Issuer and Certificate

```bash
# Create the Issuer resource
kubectl apply -f issuer.yaml

# Create the Certificate resource
vi cert.yaml
kubectl apply -f cert.yaml
```

### Step 4 — Verify the Certificate and Secret

```bash
# Check certificate status
kubectl get certificate

# Check generated secret
kubectl get secret

# Inspect the secret contents
kubectl describe secret <secret-name>
```

The secret contains three fields:
1. `tls.crt` — The signed certificate.
2. `tls.key` — The generated private key.
3. `ca.crt` — The CA certificate (if applicable).

### Step 5 — Confirm Certificate is Ready

```bash
kubectl get certificate
```

The `READY` column transitions from `False` → `True` once the Operator has successfully issued the certificate. This demonstrates the Reconciliation Loop in action — the Operator detected the desired state (certificate should be issued) and updated the actual state to match.

---

## Summary

| Concept | Description |
|---|---|
| **Operator** | Automates application lifecycle management in Kubernetes |
| **CRD** | Defines a new custom resource type |
| **CR** | An instance of a CRD representing desired state |
| **Reconciliation Loop** | Continuous process that enforces desired == actual state |
| **OLM** | Tool to install and manage Operators |
| **OperatorHub** | Public catalog of available Operators |

---

*Notes by Rahul — CKA , Day 50*