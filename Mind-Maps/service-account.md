# Kubernetes Service Account — Mind Map
```text
┌─────────────────────────────────────────┐
│          SERVICE ACCOUNT (SA)           │
└─────────────────────────────────────────┘
                    │
    ┌───────────────┼────────────────┐
    │               │                │
    ▼               ▼                ▼

IDENTITY       AUTHENTICATION      AUTHORIZATION
(Pod/User?)      (Who am I?)      (What can I do?)
    │               │                │
    │               │                │
Human User      SA Token         RBAC
(Login)         Mounted          (Role/ClusterRole)
                in Pod           +
                                 RoleBinding/
                                 ClusterRoleBinding

────────────────────────────────────────────────────────

1. PURPOSE
    │
    ├── Identity for Pods
    ├── Access Kubernetes API
    ├── Used by Applications
    ├── Used by Automation Tools
    └── Used by CI/CD Systems

Example:
Pod → ServiceAccount → API Server

────────────────────────────────────────────────────────

2. DEFAULT SERVICE ACCOUNT
    │
    ├── Created automatically
    ├── Exists in every namespace
    ├── Used if SA not specified
    └── Production: Avoid using default

Commands:
kubectl get sa
kubectl get sa -n dev

────────────────────────────────────────────────────────

3. CUSTOM SERVICE ACCOUNT
    │
    ├── app-sa
    ├── backend-sa
    ├── jenkins-sa
    ├── prometheus-sa
    └── argocd-sa

YAML:
serviceAccountName: app-sa

Benefits:
✓ Least Privilege
✓ Better Security
✓ Easy Auditing

────────────────────────────────────────────────────────

4. SERVICE ACCOUNT TOKEN
    │
    ├── Mounted into Pod
    ├── Used for API Authentication
    └── Location:

/var/run/secrets/kubernetes.io/serviceaccount/

Contains:
├── token
├── ca.crt
└── namespace

────────────────────────────────────────────────────────

5. OLD vs NEW TOKEN MODEL
    │
    ├── Old Kubernetes
    │     ├── Secret-based Token
    │     ├── Long-lived
    │     └── Less Secure
    │
    └── Modern Kubernetes
          ├── Projected Token
          ├── Short-lived
          ├── Auto Rotated
          └── More Secure

Interview Line:
Old = Static Secret Token
New = Dynamic Projected Token

────────────────────────────────────────────────────────

6. RBAC INTEGRATION
    │
    ├── ServiceAccount = Identity
    ├── Role = Permissions
    └── RoleBinding = Assignment

Flow:

SA
 │
 ▼
RoleBinding
 │
 ▼
Role
 │
 ▼
Permissions

Example:
get pods
list pods
watch pods

────────────────────────────────────────────────────────

7. TESTING SERVICE ACCOUNT
    │
    ├── Method 1
    │
    └── kubectl auth can-i

kubectl auth can-i get pods \
--as=system:serviceaccount:dev:dev-sa

    │
    ├── Method 2
    │
    └── Temporary Pod

kubectl run test ...

Verify:
kubectl get pods

────────────────────────────────────────────────────────

8. IMAGE PULL SECRET
    │
    ├── Private Registry Access
    ├── DockerHub
    ├── ECR
    ├── GHCR
    └── GitLab Registry

Flow:

Pod
 │
 ▼
ServiceAccount
 │
 ▼
imagePullSecret
 │
 ▼
Private Registry

YAML:

imagePullSecrets:
- name: regcred

────────────────────────────────────────────────────────

9. SECURITY BEST PRACTICES
    │
    ├── Dedicated SA per App
    ├── Least Privilege RBAC
    ├── Avoid cluster-admin
    ├── Avoid default SA
    ├── Rotate Credentials
    └── Disable token if unused

automountServiceAccountToken: false

────────────────────────────────────────────────────────

10. REAL-WORLD USE CASES
    │
    ├── Jenkins
    │     └── Create/Delete Pods
    │
    ├── ArgoCD
    │     └── Deploy Resources
    │
    ├── Prometheus
    │     └── Read Cluster Data
    │
    ├── Operators
    │     └── Watch Resources
    │
    └── Custom Applications
          └── Read ConfigMaps

────────────────────────────────────────────────────────

11. AWS EKS ADVANCED TOPIC
    │
    └── IRSA
          │
          ├── IAM Roles for Service Accounts
          ├── No AWS Keys in Secrets
          ├── Temporary Credentials
          └── Recommended

Flow:

Pod
 │
 ▼
ServiceAccount
 │
 ▼
IAM Role
 │
 ▼
AWS Service (S3/ECR/etc)

────────────────────────────────────────────────────────

12. INTERVIEW MEMORY TRICK

ServiceAccount = WHO ARE YOU?
Role           = WHAT CAN YOU DO?
RoleBinding    = WHO GETS WHAT?

OR

Identity → Authentication → Authorization

SA      → Token → RBAC
```

## 30-Second Interview Revision

```text
Service Account
│
├─ Identity for Pods
├─ Authenticates using Token
├─ Permissions via RBAC
├─ Default SA exists in every namespace
├─ Use custom SA in production
├─ Modern K8s uses projected short-lived tokens
├─ Can hold imagePullSecrets
├─ Used by Jenkins, ArgoCD, Prometheus
└─ IRSA in EKS for AWS access
```

This mind map covers about **90% of Service Account questions typically asked in 0–3 year DevOps, Kubernetes, CKA, and Platform Engineer interviews**.
