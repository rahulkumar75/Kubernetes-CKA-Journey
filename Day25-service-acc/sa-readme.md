## Kubernetes Service Account — Interview + Hands-on Guide (CKA / DevOps / Real World)

Since you're preparing for Kubernetes/CKA + DevOps roles, this topic is **very important** because Service Accounts are used in:

* Pods talking to Kubernetes API
* CI/CD pipelines
* Jenkins / ArgoCD / GitHub Actions auth
* Monitoring tools
* RBAC access control
* In-cluster automation

---

# 1. What is a Service Account?

A **Service Account (SA)** is a Kubernetes identity used by **applications / pods**, not humans.

### Human Users vs Service Accounts

| Human User     | Service Account    |
| -------------- | ------------------ |
| Rahul / Admin  | app-sa             |
| Used by people | Used by Pods       |
| kubectl login  | Mounted token      |
| External auth  | Kubernetes managed |

### Interview Line:

> Service Account is a non-human identity in Kubernetes used by workloads running inside the cluster to authenticate with Kubernetes API securely.

---

# 2. Why Needed?

Suppose a Pod wants to:

* Read ConfigMaps
* List Pods
* Create Jobs
* Watch Deployments

Then it needs identity + permission.

That identity = **Service Account**

---

# 3. Default Service Account

Every namespace has one default SA:

```bash
kubectl get sa
```

Output:

```bash
default
```

If pod doesn't specify SA:

```yaml
serviceAccountName: default
```

Automatically used.

---

# 4. Real World Example

Monitoring Pod wants to list all pods.

Need:

* ServiceAccount
* Role
* RoleBinding

---

# 5. Hands-on Practical (Must Know)

---

## Step 1 Create Namespace

```bash
kubectl create ns dev
```

---

## Step 2 Create Service Account

```bash
kubectl create sa app-sa -n dev
```

Check:

```bash
kubectl get sa -n dev
```

---

## Step 3 Create Pod Using SA

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: dev
spec:
  serviceAccountName: app-sa
  containers:
  - name: nginx
    image: nginx
```

Apply:

```bash
kubectl apply -f pod.yaml
```

---

## Step 4 Verify

```bash
kubectl describe pod nginx -n dev
```

See:

```bash
Service Account: app-sa
```

---

# 6. Token Mounted Inside Pod

Inside pod:

```bash
kubectl exec -it nginx -n dev -- sh
```

Check:

```bash
/var/run/secrets/kubernetes.io/serviceaccount/
```

Contains:

```bash
token
ca.crt
namespace
```

Used to authenticate API requests.

---

# 7. Important Interview Question

## Q: Can Pod access API automatically?

Yes, if SA token mounted and permissions exist.

---

# 8. RBAC with Service Account

Service Account only gives identity.

Permission comes from:

* Role + RoleBinding
  or
* ClusterRole + ClusterRoleBinding

---

## Example: Read Pods

### Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get","list","watch"]
```

---

### RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-reader
  namespace: dev
subjects:
- kind: ServiceAccount
  name: app-sa
  namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

# 9. Test Access

Run temporary pod:

```bash
kubectl run test --rm -it --image=bitnami/kubectl -n dev --serviceaccount=app-sa -- sh
```

Inside:

```bash
kubectl get pods
```

Works.

---

# 10. Important Commands

```bash
kubectl get sa
kubectl get sa -n dev
kubectl describe sa app-sa -n dev
kubectl create sa my-sa
kubectl delete sa my-sa
```

---

# 11. YAML Shortcut (Interview Favorite)

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-sa
  namespace: dev
```

---

# 12. Disable Token Mount (Security Best Practice)

```yaml
spec:
  automountServiceAccountToken: false
```

Use when pod doesn't need API access.

---

# 13. Common Interview Questions

## Q1 What is Service Account?

Identity for pod/workload.

---

## Q2 Difference between Role and ServiceAccount?

| ServiceAccount | Role       |
| -------------- | ---------- |
| Identity       | Permission |

---

## Q3 Can one SA be used by multiple Pods?

Yes.

---

## Q4 Where token stored?

```bash
/var/run/secrets/kubernetes.io/serviceaccount/
```

---

## Q5 Is default SA secure?

Usually no extra permissions, but avoid using default in production.

---

# 14. Real DevOps Use Cases

### Jenkins Agent Pod

Uses SA to create pods.

### ArgoCD

Uses SA for deployments.

### Prometheus

Uses SA to scrape cluster data.

### Custom App

Uses SA to read secrets/configmaps.

---

# 15. CKA Exam Fast Commands

Create SA:

```bash
kubectl create sa app-sa -n dev
```

Use in pod:

```yaml
serviceAccountName: app-sa
```

---

# 16. Pro Tip for Interviews

Say this:

> Service Account gives identity to workloads, and RBAC binds permissions. Together they implement least privilege access inside Kubernetes.

Strong answer.

---

# 17. Common Mistakes

❌ Using default SA everywhere
❌ Giving cluster-admin permissions
❌ Forgetting namespace in RoleBinding
❌ Not specifying serviceAccountName in pod

---

# 18. If Interviewer Gives Scenario

### "App needs to read configmaps only"

Answer:

* Create SA
* Create Role with configmap read
* Bind via RoleBinding
* Use SA in Deployment

---

# 19. Practice Task for You

Create:

* Namespace: project-devops
* SA: dev-sa
* Role: read pods
* Bind role
* Pod using dev-sa

(You were already doing RBAC, so perfect next step.)

---

# 20. Rahul Shortcut Memory Trick

```text
SA = Who are you?
Role = What can you do?
Binding = Who gets what?
```

---

If you'd like, I can also prepare **Service Account Deep Dive for 2+ years DevOps interview level** (IRSA, projected tokens, security, Jenkins/ArgoCD usage) which gives strong switch-interview advantage.
