# 🛡️ Kubernetes Pod Security Mind Map

Based on your notes 

```text
POD SECURITY IN KUBERNETES
│
├── 1. Security Context
│   │
│   ├── Purpose
│   │   ├── Privilege Control
│   │   ├── User & Group Management
│   │   └── Access Control
│   │
│   ├── Pod Level
│   │   ├── Applies to all containers
│   │   └── Can be overridden by container level
│   │
│   ├── Container Level
│   │   └── Takes precedence over pod settings
│   │
│   ├── runAsUser
│   │   ├── UID of process
│   │   ├── 0 = root
│   │   └── Best Practice → Non-root user
│   │
│   ├── runAsGroup
│   │   ├── Primary GID
│   │   └── Controls group ownership
│   │
│   ├── runAsNonRoot
│   │   ├── true → Root forbidden
│   │   ├── false → Root allowed
│   │   └── Critical Security Setting
│   │
│   ├── fsGroup
│   │   ├── Group ownership for volumes
│   │   ├── Applied to mounted volumes
│   │   └── Shared storage permissions
│   │
│   └── allowPrivilegeEscalation
│       ├── Controls sudo/su behavior
│       ├── true → Escalation allowed
│       ├── false → Escalation blocked
│       └── Recommended = false
│
├── 2. Linux Capabilities
│   │
│   ├── Purpose
│   │   └── Fine-grained root permissions
│   │
│   ├── Container-Level Only
│   │
│   ├── Common Capabilities
│   │   │
│   │   ├── NET_ADMIN
│   │   │   ├── Network interfaces
│   │   │   ├── Routing tables
│   │   │   └── Firewall rules
│   │   │
│   │   ├── SYS_ADMIN
│   │   │   ├── Mount filesystem
│   │   │   ├── Kernel changes
│   │   │   └── Highly privileged
│   │   │
│   │   └── SYS_TIME
│   │       └── Modify system clock
│   │
│   └── Capability Management
│       │
│       ├── drop
│       │   └── Remove permissions
│       │
│       ├── add
│       │   └── Grant permissions
│       │
│       └── Best Practice
│           ├── drop ALL
│           └── add only required
│
├── 3. Pod Security Standards (PSS)
│   │
│   ├── Namespace Level Security
│   │
│   ├── Privileged
│   │   ├── Lowest security
│   │   └── Almost unrestricted
│   │
│   ├── Baseline
│   │   ├── Medium security
│   │   └── Blocks common privilege escalation
│   │
│   └── Restricted
│       ├── Highest security
│       ├── Production recommendation
│       └── Enforces hardening
│
├── 4. Namespace Enforcement
│   │
│   ├── Labels
│   │   └── pod-security.kubernetes.io/*
│   │
│   ├── Modes
│   │   │
│   │   ├── enforce
│   │   │   └── Reject pod
│   │   │
│   │   ├── audit
│   │   │   └── Log violation
│   │   │
│   │   └── warn
│   │       └── Warning only
│   │
│   └── Example
│       └── enforce=restricted
│
├── 5. Restricted Policy Requirements
│   │
│   ├── runAsNonRoot=true
│   ├── allowPrivilegeEscalation=false
│   ├── seccompProfile=RuntimeDefault
│   ├── drop ALL capabilities
│   └── No dangerous capabilities
│
├── 6. seccompProfile
│   │
│   ├── Filters Linux syscalls
│   ├── Reduces attack surface
│   └── RuntimeDefault recommended
│
├── 7. Interview Scenarios
│   │
│   ├── Run pod as non-root
│   ├── Secure shared volume access
│   ├── Restrict sudo privileges
│   ├── Allow network admin capability
│   ├── Namespace hardening
│   └── Restricted namespace setup
│
└── 8. Best Practices
    │
    ├── Never run as root
    ├── runAsNonRoot=true
    ├── allowPrivilegeEscalation=false
    ├── Drop ALL capabilities
    ├── Add only needed capabilities
    ├── Use seccomp RuntimeDefault
    ├── Use Restricted PSS
    └── Follow Least Privilege Principle
```

---

# 🎯 Interview Cheat Sheet (Most Asked Questions)

### Q1. What is Security Context?

A Security Context defines privilege and access control settings for Pods and Containers.

Examples:

* runAsUser
* runAsGroup
* runAsNonRoot
* fsGroup
* allowPrivilegeEscalation
* capabilities

---

### Q2. Difference between runAsUser and runAsNonRoot?

| runAsUser     | runAsNonRoot             |
| ------------- | ------------------------ |
| Specifies UID | Ensures UID is not root  |
| Example: 1000 | Example: true            |
| Who to run as | Prevents running as root |

---

### Q3. What is fsGroup?

Used for mounted volume permissions.

All files created inside mounted volumes inherit the specified group ownership.

---

### Q4. What is allowPrivilegeEscalation?

Controls whether a process can gain additional privileges using mechanisms like sudo or su.

Recommended:

```yaml
allowPrivilegeEscalation: false
```

---

### Q5. What are Linux Capabilities?

Linux capabilities split root privileges into smaller permissions.

Examples:

| Capability | Use                   |
| ---------- | --------------------- |
| NET_ADMIN  | Networking            |
| SYS_ADMIN  | System administration |
| SYS_TIME   | Change clock          |

---

### Q6. What is the safest capability strategy?

```yaml
capabilities:
  drop:
    - ALL
```

Then explicitly add only required capabilities.

---

### Q7. What are Pod Security Standards?

Three built-in security profiles:

```text
Privileged
   ↓
Baseline
   ↓
Restricted
```

Security increases as you move downward.

---

### Q8. How do you enable Restricted mode?

```bash
kubectl create ns restricted

kubectl label ns restricted \
pod-security.kubernetes.io/enforce=restricted
```

---

### Q9. What is required by Restricted policy?

```yaml
runAsNonRoot: true

allowPrivilegeEscalation: false

seccompProfile:
  type: RuntimeDefault

capabilities:
  drop:
    - ALL
```

---

# ⚡ 30-Second Revision

```text
Security Context
├─ runAsUser
├─ runAsGroup
├─ runAsNonRoot
├─ fsGroup
├─ allowPrivilegeEscalation
└─ capabilities

Capabilities
├─ NET_ADMIN
├─ SYS_ADMIN
├─ SYS_TIME
├─ add
└─ drop ALL

Pod Security Standards
├─ Privileged
├─ Baseline
└─ Restricted

Restricted Requires
├─ Non-root
├─ No privilege escalation
├─ RuntimeDefault seccomp
└─ Drop ALL capabilities
```

This is sufficient for **CKA + DevOps/SRE + Kubernetes interviews (0–3 YOE)** and covers nearly all Pod Security questions asked in interviews.
