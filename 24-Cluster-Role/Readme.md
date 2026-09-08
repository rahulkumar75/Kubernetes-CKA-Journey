# 24 — RBAC Continued: ClusterRole and ClusterRoleBinding

## Objective

By the end of this topic, you should understand:

* Difference between `Role` and `ClusterRole`
* Difference between `RoleBinding` and `ClusterRoleBinding`
* Namespace-scoped vs cluster-scoped resources
* How to create `ClusterRole` and `ClusterRoleBinding`
* How to grant cluster-level permissions to a user
* How to verify permissions using `kubectl auth can-i`
* How to use imperative and declarative approaches

---

# 1. Role vs ClusterRole

The key difference is **scope**.

| Object               | Scope     | Typical use                                                  |
| -------------------- | --------- | ------------------------------------------------------------ |
| `Role`               | Namespace | Pods, Deployments, Services in one namespace                 |
| `ClusterRole`        | Cluster   | Nodes, namespaces, or permissions across multiple namespaces |
| `RoleBinding`        | Namespace | Assign Role/ClusterRole permissions in one namespace         |
| `ClusterRoleBinding` | Cluster   | Assign ClusterRole permissions across the cluster            |

### Memory Rule

> **Role = Namespace-level permissions**
> **ClusterRole = Cluster-level permission definition**

---

# 2. Namespace-Scoped vs Cluster-Scoped Resources

Before creating RBAC rules, determine whether the resource is namespace-scoped or cluster-scoped.

### List namespace-scoped resources

```bash
kubectl api-resources --namespaced=true
```

Examples:

```text
pods
services
deployments
configmaps
secrets
```

### List cluster-scoped resources

```bash
kubectl api-resources --namespaced=false
```

Examples:

```text
nodes
namespaces
persistentvolumes
clusterroles
clusterrolebindings
```

### Important

A **cluster-scoped resource does not belong to any namespace**.

For example:

```text
Node
  ↓
Cluster-scoped
  ↓
No namespace
```

Therefore, a namespace `Role` cannot grant access to Nodes.

---

# 3. Scenario

Suppose user `krishna` currently cannot access Nodes:

```bash
kubectl auth can-i get nodes --as=krishna
```

Expected:

```text
no
```

We want to give Krishna:

```text
get
list
watch
```

access to Nodes across the cluster.

The workflow is:

```text
ClusterRole
     ↓
Defines node permissions
     ↓
ClusterRoleBinding
     ↓
Assigns permissions to krishna
     ↓
Verify
```

---

# 4. Step 1 — Create ClusterRole

There are two approaches:

* Imperative
* Declarative

---

## Option A — Imperative

Create a ClusterRole:

```bash
kubectl create clusterrole node-reader \
  --verb=get,list,watch \
  --resource=nodes
```

Verify:

```bash
kubectl get clusterrole node-reader
```

Inspect:

```bash
kubectl describe clusterrole node-reader
```

You should see:

```text
Resources: nodes
Verbs:     get,list,watch
```

---

## Option B — Declarative

Create `node-reader.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["nodes"]
    verbs: ["get", "list", "watch"]
```

Apply:

```bash
kubectl apply -f node-reader.yaml
```

Verify:

```bash
kubectl get clusterrole node-reader
```

Inspect:

```bash
kubectl describe clusterrole node-reader
```

### Understand the rule

```yaml
apiGroups: [""]
resources: ["nodes"]
verbs: ["get", "list", "watch"]
```

Means:

```text
Core API group
      +
Nodes
      +
Read permissions
```

---

# 5. Step 2 — Create ClusterRoleBinding

The ClusterRole defines permissions, but does not give those permissions to Krishna.

Create a ClusterRoleBinding.

## Imperative

```bash
kubectl create clusterrolebinding node-reader-binding \
  --clusterrole=node-reader \
  --user=krishna
```

Verify:

```bash
kubectl get clusterrolebinding node-reader-binding
```

Inspect:

```bash
kubectl describe clusterrolebinding node-reader-binding
```

---

## Declarative

Create `node-reader-binding.yaml`:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-reader-binding
subjects:
  - kind: User
    name: krishna
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply:

```bash
kubectl apply -f node-reader-binding.yaml
```

The relationship is now:

```text
User: krishna
      ↓
ClusterRoleBinding
      ↓
ClusterRole: node-reader
      ↓
get/list/watch Nodes
```

---

# 6. Step 3 — Verify Permissions

Check before/after access:

```bash
kubectl auth can-i get nodes --as=krishna
```

Expected:

```text
yes
```

Check:

```bash
kubectl auth can-i list nodes --as=krishna
```

```text
yes
```

Check an operation that was not granted:

```bash
kubectl auth can-i delete nodes --as=krishna
```

Expected:

```text
no
```

List all permissions:

```bash
kubectl auth can-i --list --as=krishna
```

---

# 7. Test the Actual User

If Krishna has a valid kubeconfig and authentication credentials:

```bash
kubectl config get-contexts
```

Switch to Krishna's context:

```bash
kubectl config use-context krishna
```

Verify:

```bash
kubectl config current-context
```

Now:

```bash
kubectl get nodes
```

Should succeed.

Try:

```bash
kubectl delete node <node-name>
```

Should fail with:

```text
Error from server (Forbidden)
```

This confirms:

```text
Authentication → Successful
Authorization  → Based on RBAC
```

---

# 8. Important: ClusterRole Does Not Always Mean Cluster-Wide Access

This is a common CKA trap.

A `ClusterRole` defines permissions at the cluster level, but **how those permissions are bound determines the effective scope**.

### ClusterRole + ClusterRoleBinding

```text
ClusterRole
     ↓
ClusterRoleBinding
     ↓
Cluster-wide access
```

### ClusterRole + RoleBinding

A ClusterRole can also be referenced by a `RoleBinding`.

```text
ClusterRole
     ↓
RoleBinding in namespace "dev"
     ↓
Permissions limited to "dev"
```

Therefore:

> **ClusterRole = cluster-scoped permission definition**
> **ClusterRoleBinding = cluster-wide assignment**
> **RoleBinding = namespace-scoped assignment**

---

# 9. Role vs ClusterRole Example

### Role

```text
Role: pod-reader
Namespace: dev

get/list/watch Pods
       ↓
Only in dev namespace
```

### ClusterRole

```text
ClusterRole: node-reader

get/list/watch Nodes
       ↓
Can be bound cluster-wide
```

---

# 10. RoleBinding vs ClusterRoleBinding

|                           | RoleBinding      | ClusterRoleBinding  |
| ------------------------- | ---------------- | ------------------- |
| Scope                     | Namespace        | Cluster             |
| Can reference Role        | Yes              | No                  |
| Can reference ClusterRole | Yes              | Yes                 |
| Cluster-wide assignment   | No               | Yes                 |
| Typical use               | Namespace access | Cluster-wide access |

### Example

```yaml
# RoleBinding
roleRef:
  kind: Role
  name: pod-reader
```

vs.

```yaml
# ClusterRoleBinding
roleRef:
  kind: ClusterRole
  name: node-reader
```

---

# 11. Common CKA Trap

Suppose the question says:

> Give user `krishna` access to Nodes.

First ask:

```text
Is Node namespace-scoped?
        ↓
No
        ↓
Cluster-scoped
        ↓
Use ClusterRole
```

Then:

```text
ClusterRole
      ↓
ClusterRoleBinding
      ↓
User
```

### Do NOT do:

```text
Role
 ↓
RoleBinding
 ↓
Node
```

A Role is namespace-scoped and cannot grant permissions to a cluster-scoped resource such as Nodes.

---

# 12. Useful Commands

### ClusterRoles

```bash
kubectl get clusterroles
```

```bash
kubectl describe clusterrole node-reader
```

### ClusterRoleBindings

```bash
kubectl get clusterrolebindings
```

```bash
kubectl describe clusterrolebinding node-reader-binding
```

### Check resource scope

```bash
kubectl api-resources --namespaced=true
```

```bash
kubectl api-resources --namespaced=false
```

### Check permissions

```bash
kubectl auth can-i get nodes --as=krishna
```

```bash
kubectl auth can-i --list --as=krishna
```

---

# 13. CKA Decision Flow

When an RBAC question appears:

```text
What resource?
      ↓
Is it namespace-scoped?
      │
 ┌────┴─────┐
 │          │
Yes        No
 │          │
Role      ClusterRole
 │          │
 │          │
RoleBinding ClusterRoleBinding
 │          │
Namespace   Cluster-wide
```

### Example

```text
Pods
 ↓
Namespaced
 ↓
Role + RoleBinding
```

```text
Nodes
 ↓
Cluster-scoped
 ↓
ClusterRole + ClusterRoleBinding
```

---

# 14. CKA Cheat Sheet

```text
Role
  → Namespace-scoped permission definition

ClusterRole
  → Cluster-scoped permission definition

RoleBinding
  → Namespace-scoped assignment

ClusterRoleBinding
  → Cluster-wide assignment
```

### Most important commands

```bash
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false

kubectl create clusterrole node-reader \
  --verb=get,list,watch \
  --resource=nodes

kubectl create clusterrolebinding node-reader-binding \
  --clusterrole=node-reader \
  --user=krishna

kubectl auth can-i get nodes --as=krishna
kubectl auth can-i --list --as=krishna
```

### Final Memory Rule

> **Namespaced resource → Role + RoleBinding**
> **Cluster-scoped resource → ClusterRole + ClusterRoleBinding**

> **ClusterRole can also be used with a RoleBinding to grant its permissions within a specific namespace.**
