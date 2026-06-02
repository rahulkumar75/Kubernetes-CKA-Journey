## HELM (Kubernetes Package Manager) — Mind Map

```text
                                   ┌─────────────────────┐
                                   │        HELM         │
                                   │ K8s Package Manager │
                                   └──────────┬──────────┘
                                              │
        ┌─────────────────────────────────────┼─────────────────────────────────────┐
        │                                     │                                     │
        ▼                                     ▼                                     ▼

 ┌───────────────┐                  ┌────────────────┐                  ┌─────────────────┐
 │ WHAT IS HELM? │                  │ WHY USE HELM?  │                  │ CORE COMPONENTS │
 └───────┬───────┘                  └───────┬────────┘                  └────────┬────────┘
         │                                  │                                    │
         ├─ Package Manager                  ├─ Simplifies deployments            ├─ Chart
         ├─ Like apt/yum for K8s            ├─ Reusable templates                ├─ Repository
         ├─ Deploy apps with one cmd        ├─ Easy upgrades                     ├─ Release
         └─ Uses Charts                     ├─ Rollbacks                         └─ Values.yaml
                                             └─ Environment configs

                                              │
                                              ▼

                               ┌─────────────────────────┐
                               │      HELM CHART         │
                               └──────────┬──────────────┘
                                          │
                ┌─────────────────────────┼─────────────────────────┐
                │                         │                         │
                ▼                         ▼                         ▼

       Chart.yaml                 values.yaml                 templates/
       Metadata                   User variables              K8s YAML templates

                │                         │                         │
                ▼                         ▼                         ▼

         name/version            image/tag/replicas      deployment.yaml
                                                        service.yaml
                                                        ingress.yaml

─────────────────────────────────────────────────────────────────────────────

                    HELM WORKFLOW

Developer
    │
    ▼
helm install myapp chart/
    │
    ▼
Render Templates
(values.yaml + templates)
    │
    ▼
Generate Kubernetes YAML
    │
    ▼
Apply to Kubernetes Cluster
    │
    ▼
Release Created

─────────────────────────────────────────────────────────────────────────────

                    IMPORTANT COMMANDS

helm version
│
├─ Check Helm version

helm repo add
│
├─ Add chart repository

helm search repo nginx
│
├─ Search charts

helm install myapp bitnami/nginx
│
├─ Install chart

helm list
│
├─ Show releases

helm status myapp
│
├─ Release status

helm upgrade myapp chart/
│
├─ Upgrade release

helm rollback myapp 1
│
├─ Rollback release

helm uninstall myapp
│
└─ Remove release

─────────────────────────────────────────────────────────────────────────────

                     RELEASE CONCEPT

Chart
  │
  ▼
helm install
  │
  ▼
Release

Examples:
Release Name → ecommerce-prod
Chart Name   → nginx

One Chart
    ├─ Release A
    ├─ Release B
    └─ Release C

Interview:
"Chart is the package.
Release is the running instance of that package."

─────────────────────────────────────────────────────────────────────────────

                     VALUES OVERRIDE

Default values.yaml
        │
        ▼
helm install myapp chart/
        │
        ▼
Uses default configuration

OR

helm install myapp chart/ \
--set replicaCount=3

OR

helm install myapp chart/ \
-f prod-values.yaml

Interview:
"values.yaml makes charts reusable across environments."

─────────────────────────────────────────────────────────────────────────────

                      HELM UPGRADE

Version 1
     │
     ▼
helm upgrade
     │
     ▼
Version 2

Problem?
     │
     ▼
helm rollback myapp 1

Interview:
"Helm stores release history and supports rollback."

─────────────────────────────────────────────────────────────────────────────

                  HELM REPOSITORY

Repository
     │
     ├─ nginx
     ├─ mysql
     ├─ redis
     └─ prometheus

Examples:
- Bitnami
- Prometheus Community
- Grafana

Flow:
helm repo add
     │
helm repo update
     │
helm install

─────────────────────────────────────────────────────────────────────────────

                   FREQUENT INTERVIEW Q&A

Q: What is Helm?
A: Kubernetes package manager used to package, deploy, upgrade, and manage applications.

Q: What is a Chart?
A: A collection of templates and configuration files used to deploy an application.

Q: What is a Release?
A: A deployed instance of a Helm chart.

Q: Difference between Chart and Release?
A:
- Chart = Blueprint
- Release = Running deployment

Q: Where are values stored?
A: In values.yaml or passed through --set / custom values file.

Q: How do you upgrade an application?
A: helm upgrade

Q: How do you rollback?
A: helm rollback <release> <revision>

Q: Why Helm instead of raw YAML?
A:
- Reusability
- Templating
- Versioning
- Rollback support
- Easier deployments

─────────────────────────────────────────────────────────────────────────────

                  REAL-WORLD DEVOPS FLOW

Git Repo
   │
   ▼
Helm Chart
   │
   ▼
CI/CD Pipeline
(GitHub Actions/Jenkins)
   │
   ▼
helm upgrade --install
   │
   ▼
Kubernetes Cluster
(EKS / AKS / GKE / On-Prem)

```

### 30-Second Interview Summary

> **Helm is the package manager for Kubernetes.**
> It uses **Charts** (templates + configurations) to deploy applications. When a chart is installed, it creates a **Release**. Helm simplifies deployment, upgrades, rollbacks, and environment-specific configuration management through `values.yaml`. It is widely used in CI/CD pipelines for managing Kubernetes applications.
