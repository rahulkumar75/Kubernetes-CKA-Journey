# Kubernetes Admission Controllers - Mind Map (0–3 Years DevOps/CKA Interview)

```text
                          ┌───────────────────────────┐
                          │ ADMISSION CONTROLLERS     │
                          └─────────────┬─────────────┘
                                        │
        ┌───────────────────────────────┼───────────────────────────────┐
        │                               │                               │
        ▼                               ▼                               ▼

 ┌──────────────┐              ┌───────────────┐              ┌────────────────┐
 │ WHAT IS IT?  │              │ WORKING FLOW  │              │ TYPES          │
 └──────┬───────┘              └───────┬───────┘              └───────┬────────┘
        │                              │                              │
        │                              │                              │
        ▼                              ▼                              ▼

Intercepts API requests      User → API Server                  1. Mutating
BEFORE object is stored      → Authentication                   2. Validating
in etcd                      → Authorization
                             → Admission Controller
                             → etcd

        │                              │                              │
        ▼                              ▼                              ▼

Adds validation,               Can modify request          Can only approve/reject
security & defaults            before persistence          request

─────────────────────────────────────────────────────────────────────────────

                 API REQUEST FLOW


                        User
                         │
                         ▼
                        Authentication
                         │
                         ▼
                        Authorization
                         │
                         ▼
                        Mutating Admission Controller
                         │     (Modify Object)
                         ▼
                        Validating Admission Controller
                         │     (Allow/Deny)
                         ▼
                        etcd

─────────────────────────────────────────────────────────────────────────────

                    MUTATING ADMISSION CONTROLLERS

          ┌──────────────────────────────────┐
          │ Purpose: Modify Object           │
          └──────────────┬───────────────────┘
                         │
     ┌───────────────────┼────────────────────┐
     │                   │                    │
     ▼                   ▼                    ▼

DefaultStorageClass   DefaultTolerationSeconds   ServiceAccount

Adds default         Adds default pod         Mounts service account
storage class        tolerations              token automatically

Example:
Create PVC without storageClass
→ Admission Controller adds one

─────────────────────────────────────────────────────────────────────────────

                    VALIDATING ADMISSION CONTROLLERS

          ┌──────────────────────────────────┐
          │ Purpose: Allow or Reject         │
          └──────────────┬───────────────────┘
                         │
      ┌──────────────────┼──────────────────┐
      │                  │                  │
      ▼                  ▼                  ▼

ResourceQuota      LimitRanger      PodSecurity

Checks namespace   Enforces CPU/    Enforces security
resource limits    Memory limits    policies

Example:
Namespace quota = 10 Pods
11th Pod → Rejected

─────────────────────────────────────────────────────────────────────────────

                    COMMON BUILT-IN CONTROLLERS

 ┌──────────────────────────┬────────────────────────────┐
 │ Controller               │ Purpose                    │
 ├──────────────────────────┼────────────────────────────┤
 │ NamespaceLifecycle       │ Namespace validation       │
 │ LimitRanger              │ CPU/Memory limits          │
 │ ResourceQuota            │ Namespace quotas           │
 │ ServiceAccount           │ Attach SA to Pods          │
 │ PodSecurity              │ Pod security rules         │
 │ DefaultStorageClass      │ Default StorageClass       │
 │ DefaultTolerationSeconds │ Node toleration defaults   │
 │ RuntimeClass             │ Runtime validation         │
 └──────────────────────────┴────────────────────────────┘

─────────────────────────────────────────────────────────────────────────────

                    ADMISSION WEBHOOKS

                     ┌──────────────┐
                     │ WEBHOOKS     │
                     └──────┬───────┘
                            │
         ┌──────────────────┼─────────────────┐
         │                  │                 │
         ▼                  ▼                 ▼

MutatingWebhook   ValidatingWebhook   Custom Policies

Modify Pods       Approve/Deny        Organization Rules

Example:
Inject Istio Sidecar
Automatically add labels
Enforce company standards

─────────────────────────────────────────────────────────────────────────────

                    INTERVIEW QUESTIONS

Q1: What is an Admission Controller?

✔ Plugin in API Server that intercepts requests after
Authentication & Authorization and before storing in etcd.

------------------------------------------------------

Q2: Difference between Mutating and Validating?

Mutating:
✔ Can modify object
✔ Runs first

Validating:
✔ Cannot modify object
✔ Allow or reject only

------------------------------------------------------

Q3: Where do Admission Controllers run?

✔ Inside kube-apiserver

------------------------------------------------------

Q4: At which stage do they execute?

Authentication
      ↓
Authorization
      ↓
Admission Controller
      ↓
etcd

------------------------------------------------------

Q5: Example of Mutating Controller?

✔ ServiceAccount
✔ DefaultStorageClass
✔ DefaultTolerationSeconds

------------------------------------------------------

Q6: Example of Validating Controller?

✔ ResourceQuota
✔ LimitRanger
✔ PodSecurity

------------------------------------------------------

Q7: How to view enabled Admission Controllers?

kube-apiserver --enable-admission-plugins

------------------------------------------------------

Q8: Why are Admission Controllers important?

✔ Security
✔ Governance
✔ Defaulting
✔ Validation
✔ Policy Enforcement

─────────────────────────────────────────────────────────────────────────────

                    CKA INTERVIEW MEMORY TRICK

"A-A-M-V-E"

A → Authentication
A → Authorization
M → Mutating Admission Controller
V → Validating Admission Controller
E → etcd

Request Flow:
User → Auth → AuthZ → Mutate → Validate → etcd
```

# 🚀 30-Second Interview Answer

**"Admission Controllers are plugins that run inside the Kubernetes API Server. They intercept API requests after Authentication and Authorization but before objects are stored in etcd. Mutating Admission Controllers can modify objects, while Validating Admission Controllers can only approve or reject them. Common examples are ServiceAccount, DefaultStorageClass, ResourceQuota, LimitRanger, and PodSecurity. They are mainly used for security, policy enforcement, validation, and adding default configurations."**
