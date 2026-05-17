# Helm Zero to Hero

> **Quick Links:**
> - 📦 [Sample Project (best-commerce)](https://github.com/iam-veeramalla/helm-zero-to-hero/tree/main/best-commerce)
> - 🛠️ [Official Installation Guide](https://helm.sh/docs/intro/install/)
> - 📋 [Helm Cheat Sheet](https://helm.sh/docs/intro/cheatsheet/)

---

## Table of Contents

1. [What is Helm?](#what-is-helm)
2. [Core Concepts](#core-concepts)
3. [Installing Apps with Helm](#installing-apps-with-helm)
4. [Creating Your Own Helm Charts](#creating-your-own-helm-charts)
5. [Packaging & Publishing Charts](#packaging--publishing-charts)
6. [Real-World Scenario: E-Commerce Microservices](#real-world-scenario-e-commerce-microservices)
7. [Quick Reference](#quick-reference)

---

## What is Helm?

Helm is the **package manager for Kubernetes** — think of it like `apt` on Linux, but for deploying apps on a K8s cluster.

| Tool | Platform | Purpose |
|------|----------|---------|
| `apt` | Linux | Install / uninstall system packages |
| `helm` | Kubernetes | Deploy / remove apps without managing individual YAML files |

**The Problem Helm Solves:**  
Deploying an app on Kubernetes typically requires writing and managing multiple YAML files (Deployment, Service, ConfigMap, Ingress, etc.). Helm bundles all of these into a single unit called a **Chart**, so you can deploy a complex app with one command.

---

## Core Concepts

### 1. Repository
A centralised registry where Helm Charts are stored and shared.

- **Bitnami** is the most popular public repository for day-to-day use.
- You can also host your own private repo (e.g., on GitHub Pages).

### 2. Chart
A **bundle** of all the Kubernetes YAML files needed to deploy an application.  
Think of it as an installer package (like a `.deb` or `.exe` file).

```
my-app/
├── Chart.yaml        # Metadata: name, version, description
├── values.yaml       # Default configuration values
└── templates/        # Kubernetes manifest templates (Deployment, Service, etc.)
    ├── deployment.yaml
    ├── service.yaml
    └── _helpers.tpl
```

### 3. Release
A **deployed instance** of a Chart on your cluster. You give it a name at install time.

> You can deploy the same Chart multiple times (e.g., `nginx-prod`, `nginx-staging`) — each is a separate Release.

---

## Installing Apps with Helm

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

## Creating Your Own Helm Charts

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

## Packaging & Publishing Charts

Once charts are ready, bundle and share them via a repository.

### Package the Chart

```bash
helm package ./payment    # Creates payment-0.1.0.tgz
helm package ./shipping   # Creates shipping-0.1.0.tgz
```

### Generate the Repository Index

The `index.yaml` file is what makes a directory a valid Helm repository. It lists all charts and their metadata.

```bash
helm repo index .
# This creates/updates index.yaml in the current directory
```

### Host on GitHub Pages

1. Push the `.tgz` chart files and `index.yaml` to a GitHub repository.
2. Enable **GitHub Pages** on that repo (Settings → Pages → Deploy from branch).
3. Anyone can now add your repo:

```bash
helm repo add my-org https://<username>.github.io/<repo-name>
helm install payment-service my-org/payment
```

---

## Real-World Scenario: E-Commerce Microservices

**Company:** BestCommerce  
**Microservices:** `payment`, `shipping`

### Workflow

```
1. Create chart scaffolds
   helm create payment
   helm create shipping

2. Customise Chart.yaml, values.yaml, and templates/ for each service

3. Package both charts
   helm package ./payment
   helm package ./shipping

4. Generate repo index
   helm repo index .

5. Push to GitHub → Enable GitHub Pages

6. Deploy to cluster
   helm repo add bestcommerce https://bestcommerce.github.io/helm-charts
   helm install payment-svc bestcommerce/payment
   helm install shipping-svc bestcommerce/shipping
```

---

## Quick Reference

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
helm search hub <keyword>         # Search on Artifact Hub (public)
```

### Install / Upgrade / Rollback

```bash
helm install <release> <chart>              # Fresh install
helm upgrade <release> <chart>              # Upgrade existing release
helm upgrade --install <release> <chart>    # Install if not exists, else upgrade
helm rollback <release> <revision>          # Roll back to a previous revision
```

### Inspect

```bash
helm list                         # List all releases
helm status <release>             # Status of a release
helm history <release>            # Revision history
helm show values <chart>          # Show default values of a chart
helm get values <release>         # Show values used in a deployed release
helm template <release> <chart>   # Render templates locally (dry-run)
```

### Package & Publish

```bash
helm create <name>                # Scaffold a new chart
helm package <chart-dir>          # Package chart into .tgz
helm repo index <dir>             # Generate index.yaml for a repo
helm lint <chart-dir>             # Validate chart for errors
```

### Uninstall

```bash
helm uninstall <release>          # Remove a release from the cluster
```

---
