# Helm Zero to Hero

> **Quick Links:**
> - 📦 [Sample Project (best-commerce)](https://github.com/iam-veeramalla/helm-zero-to-hero/tree/main/best-commerce)
> - 🛠️ [Official Installation Guide](https://helm.sh/docs/intro/install/)
> - 📋 [Helm Cheat Sheet](https://helm.sh/docs/intro/cheatsheet/)

---

## Table of Contents

1. [What is Helm?](#1-what-is-helm)
2. [Core Concepts](#2-core-concepts)
3. [Installing Apps with Helm](#3-installing-apps-with-helm)
4. [Creating Your Own Helm Charts](#4-creating-your-own-helm-charts)
5. [Packaging & Publishing Charts on GitHub Pages](#5-packaging--publishing-charts-on-github-pages)
6. [Real-World Scenario: E-Commerce Microservices](#6-real-world-scenario-e-commerce-microservices)
7. [Upgrades, Rollbacks & Release History](#7-upgrades-rollbacks--release-history)
8. [Quick Reference](#8-quick-reference)

---

## 1. What is Helm?

Helm is the **package manager for Kubernetes** — think of it like `apt` on Linux, but for deploying apps on a K8s cluster.

| Tool  | Platform   | Purpose                                                       |
|-------|------------|---------------------------------------------------------------|
| `apt` | Linux      | Install / uninstall system packages                           |
| `helm`| Kubernetes | Deploy / remove apps without managing individual YAML files   |

**The Problem Helm Solves:**
Deploying an app on Kubernetes typically requires writing and managing multiple YAML files (Deployment, Service, ConfigMap, Ingress, etc.). Helm bundles all of these into a single unit called a **Chart**, so you can deploy a complex app with one command.

---

## 2. Core Concepts

### Repository
A centralised registry where Helm Charts are stored and shared.
- **Bitnami** is the most popular public repository for day-to-day use.
- You can also host your own private repo (e.g., on GitHub Pages).

### Chart
A **bundle** of all the Kubernetes YAML files needed to deploy an application. Think of it as an installer package (like a `.deb` or `.exe` file).

```
my-app/
├── Chart.yaml        # Metadata: name, version, description
├── values.yaml       # Default configuration values
└── templates/        # Kubernetes manifest templates
    ├── deployment.yaml
    ├── service.yaml
    └── _helpers.tpl
```

### Release
A **deployed instance** of a Chart on your cluster. You give it a name at install time.

> You can deploy the same Chart multiple times (e.g., `nginx-prod`, `nginx-staging`) — each is a separate Release.

---

## 3. Installing Apps with Helm

### Prerequisites
- `kubectl` configured and pointing to the correct cluster.
- Verify your current context before running any Helm commands:

```bash
kubectl config current-context
```

### 3-Step Installation Process

#### Step 1 — Add the Repository

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update   # Always update after adding a new repo
```

#### Step 2 — Search for the App

```bash
helm search repo bitnami | grep nginx
# or more specifically:
helm search repo bitnami/nginx
```

#### Step 3 — Install the App

```bash
helm install <release-name> bitnami/nginx
# Example:
helm install my-nginx bitnami/nginx
```

### Verify Installation

```bash
kubectl get pods
kubectl get svc
helm list             # Lists all installed releases
```

### Installing from a Non-Bitnami Source

If the app isn't in Bitnami, find its chart repository from the project's documentation, then:

```bash
helm repo add <repo-name> <repo-url>
helm install <release-name> <repo-name>/<chart-name>
```

### Uninstalling an App

```bash
helm uninstall <release-name>
# Example:
helm uninstall my-nginx
```

> ⚠️ You must provide the **Release name** (not the chart name) when uninstalling.

---

## 4. Creating Your Own Helm Charts

As a DevOps engineer, you'll often need to:
- Package your own application as a Helm Chart.
- Share it with the team via a private repository.
- Deploy it consistently across environments.

### Step 1 — Scaffold a New Chart

```bash
helm create <chart-name>
# Example:
helm create payment
```

This generates the full directory structure automatically.

### Step 2 — Understand the Key Files

#### `Chart.yaml` — Metadata

```yaml
apiVersion: v2
name: payment
description: A Helm chart for the Payment microservice
type: application
version: 0.1.0       # Chart version (update on every chart change)
appVersion: "1.0.0"  # App image version
```

#### `values.yaml` — Default Configuration

This is where you define the default values for your templates. Users can override these at install time.

```yaml
replicaCount: 1

image:
  repository: my-org/payment-service
  tag: "1.0.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
```

#### `templates/deployment.yaml` — Templated Manifest

Templates use Go templating syntax (`{{ }}`) to reference values from `values.yaml` instead of hardcoding them.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-payment
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
        - name: payment
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

> **Why templates?** The same chart can be deployed in `dev`, `staging`, and `prod` just by changing the values — no need to maintain separate YAML files per environment.

### Step 3 — Inspect a Chart's Default Values

```bash
helm show values <chart-name>
```

### Step 4 — Install Your Local Chart

```bash
helm install <release-name> ./<chart-directory>
# Example:
helm install payment-service ./payment
```

### Step 5 — Override Values at Install Time

```bash
# Using --set flag (quick, for single values)
helm install payment-service ./payment --set replicaCount=3

# Using a custom values file (recommended for multiple overrides)
helm install payment-service ./payment -f custom-values.yaml
```

---

## 5. Packaging & Publishing Charts on GitHub Pages

Once charts are ready, bundle and share them via a repository.

### Correct Repository Structure

> ⚠️ **Common Mistake:** GitHub Pages only exposes files from the branch root (or selected folder). Your `index.yaml` must be at the correct path.

**Wrong ❌**
```
helm-chart branch
└── Helm-Chart/
    └── best-commerce/
        └── charts/
            └── index.yaml    ← Pages can't see this
```

**Correct ✅**
```
helm-chart branch
└── charts/
    ├── index.yaml
    ├── payments-0.1.0.tgz
    ├── orders-0.1.0.tgz
    └── shipping-0.1.0.tgz
```

---

### Step-by-Step: Publish to GitHub Pages

#### Step 1 — Use a Dedicated Branch

```bash
git checkout helm-chart
mkdir -p charts
```

#### Step 2 — Package Your Charts

```bash
helm package payments  -d charts
helm package orders    -d charts
helm package shipping  -d charts
```

Result:
```
charts/
├── payments-0.1.0.tgz
├── orders-0.1.0.tgz
└── shipping-0.1.0.tgz
```

#### Step 3 — Generate `index.yaml`

Run from repo root:

```bash
helm repo index charts \
  --url https://rahulkumar75.github.io/Kubernetes-CKA/charts
```

This creates `charts/index.yaml` — the file that makes the directory a valid Helm repo.

#### Step 4 — Commit & Push

```bash
git add .
git commit -m "Add helm repository"
git push origin helm-chart
```

#### Step 5 — Configure GitHub Pages

Go to: **Repo → Settings → Pages**

| Setting | Value              |
|---------|--------------------|
| Source  | Deploy from branch |
| Branch  | `helm-chart`       |
| Folder  | `/ (root)`         |

Save and wait 1–2 minutes.

#### Step 6 — Verify

Open in browser:
```
https://rahulkumar75.github.io/Kubernetes-CKA/charts/index.yaml
```

If it loads → ✅ Success.

#### Step 7 — Add & Use Your Helm Repo

```bash
helm repo add my-org https://rahulkumar75.github.io/Kubernetes-CKA/charts
helm repo update
helm search repo my-org

# Install a chart
helm install payments-release my-org/payments
```

#### Troubleshooting: `index.yaml` gives 404

Your `charts/` folder is in the wrong place. It must be at the **branch root**, not nested inside subdirectories.

---

## 6. Real-World Scenario: E-Commerce Microservices

**Company:** BestCommerce
**Microservices:** `payment`, `shipping`

### Full Workflow

```bash
# 1. Create chart scaffolds
helm create payment
helm create shipping

# 2. Customise Chart.yaml, values.yaml, and templates/ for each service

# 3. Package both charts
helm package ./payment  -d charts
helm package ./shipping -d charts

# 4. Generate repo index
helm repo index charts \
  --url https://bestcommerce.github.io/helm-charts/charts

# 5. Push to GitHub → Enable GitHub Pages

# 6. Add repo and deploy to cluster
helm repo add bestcommerce https://bestcommerce.github.io/helm-charts/charts
helm repo update
helm install payment-svc  bestcommerce/payment
helm install shipping-svc bestcommerce/shipping
```

---

## 7. Upgrades, Rollbacks & Release History

This is one of Helm's biggest advantages in DevOps CI/CD pipelines.

### The Full Lifecycle

#### 1 — Initial Install

```bash
helm install payments-release my-org/payments
# → Creates Revision 1
```

#### 2 — Upgrade the Release

After changing image version, replicas, env vars, or resources:

```bash
helm upgrade payments-release my-org/payments
# → Creates Revision 2
```

You can also upgrade with value overrides:

```bash
helm upgrade payments-release my-org/payments --set replicaCount=3
helm upgrade payments-release my-org/payments -f prod-values.yaml
```

#### 3 — Something Breaks ❌

```bash
kubectl get pods
# NAME                        READY   STATUS             RESTARTS
# payments-xyz-abc            0/1     CrashLoopBackOff   3
```

#### 4 — Check Release History

```bash
helm history payments-release
```

Example output:

```
REVISION  STATUS      DESCRIPTION
1         superseded  Install complete
2         deployed    Upgrade complete
```

#### 5 — Roll Back

```bash
helm rollback <release-name> <revision-number>

# Example: go back to revision 1
helm rollback payments-release 1
```

Helm restores:
- Old Deployment spec
- Old ConfigMaps & Secrets
- Old `values.yaml` overrides
- Old image tag
- All old manifests

Cluster returns to a stable state instantly.

---

### How Helm Stores History

Helm stores every revision as a Kubernetes Secret:

```bash
kubectl get secrets
```

```
NAME                                      TYPE
sh.helm.release.v1.payments-release.v1   helm.sh/release.v1
sh.helm.release.v1.payments-release.v2   helm.sh/release.v1
```

> **Interview Point:** This is how `helm rollback` works — it replays the stored manifests from a previous revision's secret.

---

### Mental Model

```
git revert    →  source code rollback
helm rollback →  Kubernetes deployment rollback
```

---

### Common Real-World Pattern

```bash
# Production deployment fails after upgrade
helm upgrade payments-release my-org/payments

# Immediate recovery
helm rollback payments-release 1
```

---

## 8. Quick Reference

### Repository Commands

```bash
helm repo add <name> <url>       # Add a new repository
helm repo update                  # Refresh repository metadata
helm repo list                    # List all added repositories
helm repo remove <name>           # Remove a repository
```

### Search Commands

```bash
helm search repo <keyword>        # Search in added repositories
helm search hub  <keyword>        # Search on Artifact Hub (public)
```

### Install / Upgrade / Rollback

```bash
helm install <release> <chart>                      # Fresh install
helm install <release> <chart> --set key=val        # Install with value override
helm install <release> <chart> -f values.yaml       # Install with values file
helm install my-org/payments --generate-name        # Auto-generate release name
helm upgrade <release> <chart>                      # Upgrade existing release
helm upgrade --install <release> <chart>            # Install if not exists, else upgrade
helm rollback <release> <revision>                  # Roll back to a previous revision
```

### Inspect & Debug

```bash
helm list                         # List all releases
helm status   <release>           # Status of a release
helm history  <release>           # Revision history
helm show values <chart>          # Show default values of a chart
helm get values  <release>        # Show values used in a deployed release
helm template <release> <chart>   # Render templates locally (dry-run)
helm lint     <chart-dir>         # Validate chart for errors
```

### Package & Publish

```bash
helm create   <name>              # Scaffold a new chart
helm package  <chart-dir>         # Package chart into .tgz
helm package  <chart-dir> -d <dir> # Package into a specific directory
helm repo index <dir> --url <url> # Generate index.yaml for a repo
```

### Uninstall

```bash
helm uninstall <release>          # Remove a release from the cluster
```