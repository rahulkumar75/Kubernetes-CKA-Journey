# 23 — Role-Based Access Control (RBAC)

## Objective

By the end of this topic, you should understand:

* What RBAC is and why Kubernetes uses it
* Difference between **Role** and **RoleBinding**
* How to define permissions using **verbs** and **resources**
* How to grant permissions to a user
* How to verify permissions with `kubectl auth can-i`
* How to create a kubeconfig/context for a user
* How to troubleshoot **authentication vs authorization** issues

---

# 1. What is RBAC?

**RBAC (Role-Based Access Control)** controls **what an authenticated identity is allowed to do** in Kubernetes.

The basic flow is:

```text
User
  ↓
Authentication
  ↓
Who are you?
  ↓
Authorization
  ↓
RBAC
  ↓
What can you do?
```

### Example

Suppose user `adam` needs read-only access to Pods.

```text
adam
  ↓
RoleBinding
  ↓
pod-reader Role
  ↓
get / list / watch Pods
```

### Memory Rule

> **Role = What permissions?**
> **RoleBinding = Who gets those permissions?**

---

# 2. RBAC Building Blocks

The main RBAC objects are:

```text
Role
RoleBinding

ClusterRole
ClusterRoleBinding
```

### Role vs ClusterRole

| Object        | Scope         |
| ------------- | ------------- |
| `Role`        | One namespace |
| `ClusterRole` | Cluster-wide  |

A `Role` can grant permissions only within its namespace.

A `ClusterRole` can define permissions that can be used cluster-wide or, through a `RoleBinding`, within a particular namespace.

---

# 3. RBAC Permissions

RBAC permissions are defined using:

### Resources

Examples:

```text
pods
deployments
services
configmaps
secrets
```

### Verbs

Common verbs:

```text
get
list
watch
create
update
patch
delete
```

A simple classification:

```text
get/list/watch
      ↓
    Read

create/update/patch/delete
      ↓
    Write
```

> Kubernetes RBAC is **additive**: permissions are granted; there is no "deny" rule in a Role.

---

# 4. API Resources

Before creating a Role, you should know the correct resource name and API group.

Useful command:

```bash
kubectl api-resources
```

For detailed information:

```bash
kubectl explain role
```

```bash
kubectl explain role.rules
```

```bash
kubectl explain role.rules.resources
```

### API Groups

Kubernetes APIs are organized into groups.

The **core API group** has no group name:

```yaml
apiVersion: v1
```

Examples:

```text
pods
services
configmaps
```

Named API groups look like:

```yaml
apiVersion: apps/v1
```

Examples:

```text
deployments
statefulsets
daemonsets
```

---

# 5. Hands-on Scenario

### Requirement

Create a user called `adam`.

Adam should be able to:

```text
get Pods
list Pods
watch Pods
```

But Adam should **not** be able to:

```text
create Pods
delete Pods
```

We will implement:

```text
Role
  ↓
RoleBinding
  ↓
adam
  ↓
Verify permissions
  ↓
Configure kubeconfig
  ↓
Test as adam
```

---

# 6. Step 1 — Check Current Permissions

Before creating anything:

```bash
kubectl auth can-i get pods --as=adam
```

Expected:

```text
no
```

Check another operation:

```bash
kubectl auth can-i delete pods --as=adam
```

Expected:

```text
no
```

### Important

`--as=adam` means **impersonate the user `adam`**.

It does not log you into the cluster as Adam.

You need sufficient permission to impersonate users.

---

# 7. Step 2 — Create the Role

Create `pod-reader-role.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

Apply:

```bash
kubectl apply -f pod-reader-role.yaml
```

Verify:

```bash
kubectl get role -n default
```

Inspect:

```bash
kubectl describe role pod-reader -n default
```

### Understand the rule

```yaml
apiGroups: [""]
resources: ["pods"]
verbs: ["get", "list", "watch"]
```

Means:

```text
Core API group
     +
Pods
     +
Read permissions
```

At this point:

> **The Role exists, but Adam still does not have the permissions.**

---

# 8. Step 3 — Create the RoleBinding

A Role must be bound to a user.

Create `pod-reader-binding.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: default
subjects:
  - kind: User
    name: adam
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f pod-reader-binding.yaml
```

Verify:

```bash
kubectl get rolebinding -n default
```

Inspect:

```bash
kubectl describe rolebinding pod-reader-binding -n default
```

Now the relationship is:

```text
User: adam
     ↓
RoleBinding
     ↓
Role: pod-reader
     ↓
get/list/watch Pods
```

---

# 9. Step 4 — Verify RBAC

Check read access:

```bash
kubectl auth can-i get pods --as=adam
```

```text
yes
```

```bash
kubectl auth can-i list pods --as=adam
```

```text
yes
```

Check write access:

```bash
kubectl auth can-i create pods --as=adam
```

```text
no
```

```bash
kubectl auth can-i delete pods --as=adam
```

```text
no
```

### Best verification command

```bash
kubectl auth can-i --list --as=adam
```

This shows the permissions available to the impersonated identity.

---

# 10. Step 5 — Create kubeconfig Credentials

RBAC only defines permissions.

Adam still needs valid authentication credentials.

If Adam has a signed client certificate:

```text
adam.crt
adam.key
```

Configure credentials:

```bash
kubectl config set-credentials adam \
  --client-certificate=adam.crt \
  --client-key=adam.key \
  --embed-certs=true
```

This creates the `adam` user entry in kubeconfig.

### Important

The kubeconfig `user.name` is a local configuration label.

The actual Kubernetes identity comes from the authentication credential.

For a client certificate:

```text
Certificate
    ↓
Subject / CN
    ↓
Authenticated identity
```

For example:

```text
CN=adam
    ↓
username = adam
```

---

# 11. Step 6 — Create a Context

Create a context connecting Adam's credentials to the cluster:

```bash
kubectl config set-context adam \
  --cluster=<cluster-name> \
  --user=adam \
  --namespace=default
```

Check contexts:

```bash
kubectl config get-contexts
```

Switch:

```bash
kubectl config use-context adam
```

Check:

```bash
kubectl config current-context
```

---

# 12. Step 7 — Test as Adam

Run:

```bash
kubectl get pods
```

This should work.

Try an operation Adam is not allowed to perform:

```bash
kubectl delete pod <pod-name>
```

Expected:

```text
Error from server (Forbidden)
```

This confirms:

```text
Authentication → Successful
Authorization  → Denied
```

### Very Important Troubleshooting Distinction

```text
Unauthorized
    ↓
Authentication problem

Forbidden
    ↓
Authorization / RBAC problem
```

---

# 13. RoleBinding `roleRef` is Immutable

The `roleRef` in a RoleBinding cannot be changed after the RoleBinding is created.

For example:

```yaml
roleRef:
  kind: Role
  name: pod-reader
```

You cannot simply modify it to:

```yaml
name: admin-role
```

Instead:

```text
Delete old RoleBinding
        ↓
Create new RoleBinding
```

### Why?

It prevents an existing binding from being silently redirected to a different Role.

---

# 14. Role vs RoleBinding — Quick Example

```text
Role
┌─────────────────────────┐
│ pod-reader              │
│                         │
│ get pods                │
│ list pods               │
│ watch pods              │
└─────────────────────────┘
            ↑
            │
     RoleBinding
            │
            ↓
         adam
```

Remember:

> **Role defines permissions; RoleBinding assigns them.**

---

# 15. Useful RBAC Commands

### List Roles

```bash
kubectl get roles
```

All namespaces:

```bash
kubectl get roles -A
```

### Count Roles

```bash
kubectl get roles -A --no-headers | wc -l
```

### List RoleBindings

```bash
kubectl get rolebindings -A
```

### List ClusterRoles

```bash
kubectl get clusterroles
```

### List ClusterRoleBindings

```bash
kubectl get clusterrolebindings
```

### Check permission

```bash
kubectl auth can-i get pods --as=adam
```

### List all permissions

```bash
kubectl auth can-i --list --as=adam
```

---

# 16. Troubleshooting

## Case 1 — `Forbidden`

Example:

```text
Error from server (Forbidden)
```

Authentication succeeded, but authorization failed.

Check:

```bash
kubectl auth can-i get pods --as=adam
```

Then inspect:

```bash
kubectl get role -n default
kubectl get rolebinding -n default
kubectl describe rolebinding pod-reader-binding -n default
```

Check:

* Correct username
* Correct namespace
* Correct Role
* Correct RoleBinding
* Correct resource
* Correct verbs

---

## Case 2 — `Unauthorized`

Example:

```text
You must be logged in to the server
```

This is usually an **authentication** problem.

Check:

```bash
kubectl config current-context
kubectl config view
```

Check certificate:

```bash
openssl x509 -in adam.crt -noout -subject -issuer -dates
```

Look for an expired certificate.

---

## Case 3 — Certificate Expired

Check:

```bash
openssl x509 \
  -in adam.crt \
  -noout -dates
```

If expired, issue a new certificate through the appropriate Kubernetes CA/CSR workflow.

> Do not manually copy or expose the cluster CA private key just to fix a user certificate.

For a normal Kubernetes setup, follow the CSR workflow:

```text
Generate key
    ↓
Generate CSR
    ↓
Submit Kubernetes CSR
    ↓
Approve
    ↓
Retrieve certificate
    ↓
Update kubeconfig
```

---

# 17. CKA Exam Workflow

When asked:

> **"Give user X access to resource Y."**

Think:

```text
1. Identify user
       ↓
2. Determine namespace
       ↓
3. Create Role
       ↓
4. Define resources + verbs
       ↓
5. Create RoleBinding
       ↓
6. Verify with auth can-i
```

If the task also requires the user to actually connect:

```text
7. Configure certificate credentials
       ↓
8. Create context
       ↓
9. Switch context
       ↓
10. Test kubectl
```

---
