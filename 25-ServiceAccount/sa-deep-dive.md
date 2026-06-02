## Kubernetes Service Account — Deep Dive for 2+ Years DevOps Interview Level

Since you're already working as a Junior DevOps Engineer and preparing for growth roles, this is the **real interview-level version**: not just definitions, but how Service Accounts are used in production.

---

# 1. Strong Interview Answer (2+ Years)

> A Kubernetes Service Account is an in-cluster workload identity used by pods to authenticate to the Kubernetes API or integrated cloud services. It is commonly combined with RBAC for least-privilege access and can be extended to cloud IAM roles such as IRSA in AWS.

That answer sounds experienced.

---

# 2. Real Architecture Understanding

When a Pod starts with:

```yaml
serviceAccountName: app-sa
```

Kubernetes typically provides credentials to the pod so the application can authenticate.

Flow:

```text
Pod -> Service Account Token -> Kubernetes API Server
```

Then API Server checks:

```text
Who are you? -> ServiceAccount
What can you do? -> RBAC
```

---

# 3. Default vs Custom Service Accounts

### Default SA

Every namespace gets:

```bash
default
```

If no SA specified, pod uses it.

### Production Best Practice

Never rely on default SA for apps.

Use dedicated identities:

```text
frontend-sa
backend-sa
jenkins-sa
prometheus-sa
argocd-sa
```

---

# 4. Why Custom SA in Production?

Because each workload should have separate permissions.

Example:

| App        | Access             |
| ---------- | ------------------ |
| frontend   | no API access      |
| backend    | read configmaps    |
| prometheus | list nodes/pods    |
| jenkins    | create deployments |

This is **least privilege**.

---

# 5. Token Changes (Modern Kubernetes)

Older Kubernetes created long-lived secret tokens.

Modern Kubernetes uses **projected short-lived tokens** (recommended).

Interview line:

> Newer Kubernetes versions use rotating bound service account tokens instead of static secret-based tokens.

Good advanced answer.

---

# 6. How to Check Inside Pod

```bash
kubectl exec -it pod-name -- sh
```

Check:

```bash
ls /var/run/secrets/kubernetes.io/serviceaccount/
```

Usually:

```text
token
ca.crt
namespace
```

---

# 7. Real DevOps Use Cases

## Jenkins on Kubernetes

Jenkins agent may need:

* create pods
* delete pods
* read logs

Use `jenkins-sa`

---

## Argo CD

Needs permissions to deploy resources.

Use `argocd-application-controller` SA.

---

## Prometheus

Needs read access to:

* nodes
* services
* endpoints
* pods

---

## Custom Internal App

App may watch ConfigMaps or Secrets.

---

# 8. Security Best Practices

## Disable token if not needed

```yaml
automountServiceAccountToken: false
```

Use for apps that never call Kubernetes API.

---

## Separate SA per application

Avoid one SA for all apps.

---

## Minimal RBAC

Don't give:

```text
cluster-admin
```

unless truly required.

---

# 9. Advanced RBAC Example

Backend app only reads ConfigMaps:

```yaml
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get","list"]
```

Then bind only backend SA.

---

# 10. Common Production Mistakes

## Mistake 1

Using default SA for every deployment.

## Mistake 2

Giving cluster-admin to CI/CD.

## Mistake 3

Sharing one SA across many namespaces.

## Mistake 4

Leaving unused tokens mounted.

---

# 11. AWS EKS Interview Gold Topic — IRSA

Since you know AWS, this matters.

## IRSA = IAM Roles for Service Accounts

Map Kubernetes Service Account to AWS IAM Role.

Example:

```text
backup pod -> S3 access
external-dns -> Route53 access
cluster-autoscaler -> ASG access
```

Instead of storing AWS keys in secrets.

Strong line:

> In EKS, I’d use IRSA so pods assume IAM roles securely without static credentials.

That sounds mid-level.

---

# 12. Example IRSA Concept

```text
Pod
 -> ServiceAccount annotated with IAM Role ARN
 -> OIDC trust
 -> Temporary AWS credentials
```

---

# 13. Troubleshooting Questions

## Pod gets Forbidden error

Check:

```bash
kubectl auth can-i list pods \
--as=system:serviceaccount:dev:app-sa -n dev
```

Excellent command for interview.

---

## Which SA pod uses?

```bash
kubectl get pod pod1 -o jsonpath='{.spec.serviceAccountName}'
```

---

## Describe SA

```bash
kubectl describe sa app-sa -n dev
```

---

# 14. Scenario-Based Interview Answers

## Q: Jenkins needs to deploy apps.

Answer:

* Create dedicated SA
* Bind only required RBAC
* Use namespace-scoped access where possible
* Avoid cluster-admin unless required

---

## Q: App doesn’t need Kubernetes API.

Answer:

* Disable automount token
* Run with dedicated SA or no token mount

---

## Q: Pod needs AWS S3 access in EKS.

Answer:

* Use IRSA with dedicated IAM role

---

# 15. Hands-on Practice

## Create SA

```bash
kubectl create sa backend-sa -n prod
```

## Use in deployment

```yaml
spec:
  serviceAccountName: backend-sa
```

## Verify

```bash
kubectl get deploy backend -o yaml | grep serviceAccountName
```

---

# 16. Powerful Interview Closing Statement

> Service Accounts are the identity layer for workloads. In production I prefer dedicated service accounts, minimal RBAC, disabled token mounts where unnecessary, and cloud-native identity integrations like IRSA.

Very strong.

---

# 17. For Your Career Growth

Since you want advancement roles (not monitoring-heavy), master these topics:

* Service Accounts
* RBAC
* IRSA / Workload Identity
* Admission Controllers
* Network Policies
* GitOps security
* Multi-cluster auth

These are platform engineer / senior DevOps topics.

---
