# RBAC (Role-Based Access Control) – Mind Map

```text
┌─────────────────────────────────────────────┐
│                   RBAC                      │
│       (Role Based Access Control)           │
└─────────────────────────────────────────────┘
                     │
     ┌───────────────┼───────────────┐
     │               │               │
     ▼               ▼               ▼

 Authentication   Authorization    Admission
 (Who are you?)  (What can you do?) (Final check)

     │
     ▼
 User / ServiceAccount
     │
     ▼
 Certificate / Token


═══════════════════════════════════════════════
1. RBAC OBJECTS
═══════════════════════════════════════════════

RBAC
 │
 ├── Role
 │     ├─ Namespace scoped
 │     ├─ Defines permissions
 │     └─ Example:
 │         pods → get,list,watch
 │
 ├── RoleBinding
 │     ├─ Namespace scoped
 │     ├─ Assigns Role
 │     └─ To User / Group / SA
 │
 ├── ClusterRole
 │     ├─ Cluster-wide
 │     ├─ Nodes
 │     ├─ PVs
 │     ├─ Namespaces
 │     └─ All namespaces access
 │
 └── ClusterRoleBinding
       ├─ Cluster-wide
       └─ Assigns ClusterRole


═══════════════════════════════════════════════
2. ACCESS FLOW
═══════════════════════════════════════════════

User Rahul
      │
      ▼
RoleBinding
      │
      ▼
Role
      │
      ▼
Permissions Granted

Example:

Rahul
  │
  ▼
RoleBinding
  │
  ▼
pod-reader
  │
  ▼
get,list,watch pods


═══════════════════════════════════════════════
3. COMMON VERBS
═══════════════════════════════════════════════

Read Access
│
├─ get
├─ list
└─ watch

Write Access
│
├─ create
├─ update
├─ patch
└─ delete

Admin Access
│
├─ *
└─ all resources


═══════════════════════════════════════════════
4. SUBJECTS
═══════════════════════════════════════════════

RoleBinding
    │
    ├── User
    │      Rahul
    │
    ├── Group
    │      Developers
    │
    └── ServiceAccount
           app-sa


═══════════════════════════════════════════════
5. ROLE vs CLUSTERROLE
═══════════════════════════════════════════════

Role
│
├─ Namespace only
├─ Pods
├─ Services
└─ Deployments

ClusterRole
│
├─ Entire cluster
├─ Nodes
├─ PVs
├─ Namespaces
└─ Cross-namespace access


═══════════════════════════════════════════════
6. ROLEBINDING vs CLUSTERROLEBINDING
═══════════════════════════════════════════════

RoleBinding
│
├─ Namespace scoped
└─ Binds Role/ClusterRole
   inside one namespace

ClusterRoleBinding
│
├─ Cluster-wide
└─ Binds ClusterRole
   across cluster


═══════════════════════════════════════════════
7. MOST USED COMMANDS
═══════════════════════════════════════════════

Create Role
kubectl create role

View Roles
kubectl get roles

View RoleBindings
kubectl get rolebindings

Check Permissions
kubectl auth can-i get pods

Describe Role
kubectl describe role <name>

Switch User Context
kubectl config use-context <ctx>


═══════════════════════════════════════════════
8. INTERVIEW QUESTIONS
═══════════════════════════════════════════════

Q. What is RBAC?
→ Kubernetes authorization mechanism.

Q. Difference between Role & ClusterRole?
→ Namespace vs Cluster-wide.

Q. Difference between RoleBinding &
   ClusterRoleBinding?
→ Namespace vs Cluster-wide binding.

Q. How to verify permissions?
→ kubectl auth can-i

Q. Can a Role grant cluster access?
→ No.


═══════════════════════════════════════════════
9. CKA EXAM WORKFLOW
═══════════════════════════════════════════════

Create Namespace
        │
        ▼
Create Role
        │
        ▼
Create RoleBinding
        │
        ▼
Switch User Context
        │
        ▼
kubectl auth can-i
        │
        ▼
Verify Access
```

### 30-Second Revision

```text
RBAC
│
├─ Role            → Permissions in Namespace
├─ RoleBinding     → Assign Role
├─ ClusterRole     → Cluster-wide Permissions
├─ ClusterRoleBinding → Assign ClusterRole
│
├─ Subjects:
│   ├─ User
│   ├─ Group
│   └─ ServiceAccount
│
└─ Verify:
    kubectl auth can-i <verb> <resource>
```

For a **0–3 year DevOps/Kubernetes interview**, knowing the above plus one hands-on example (Role + RoleBinding for pod-reader) is usually sufficient.
