# Pod Security in Kubernetes

## Overview

Pod security in Kubernetes involves two primary components:

1. **Security Context** — Defines privilege and access control settings for pods and containers.
2. **Linux Capabilities** — Fine-grained controls over what a container process is allowed to do, defined *within* the Security Context.

---

## Security Context

A Security Context defines privilege and access control settings at both the **Pod level** and the **Container level**. Container-level settings override Pod-level settings when both are specified.

### `runAsUser`

Specifies the **user ID (UID)** under which the container process runs.

- UID `0` = root user
- UID `1000` = a non-root user with ID 1000

```yaml
securityContext:
  runAsUser: 1000
```

> **Best practice:** Avoid running containers as root (UID 0) unless absolutely necessary.

---

### `runAsGroup`

Specifies the **primary group ID (GID)** for the container process.

```yaml
securityContext:
  runAsGroup: 3000
```

Multiple users can belong to groups like `developers`, `admins`, or `devops`. This field controls which group the container process is associated with.

---

### `runAsNonRoot`

A boolean that enforces the container must **not** run as root.

| Value | Behavior |
|-------|----------|
| `true` | Container is rejected if it tries to run as root |
| `false` | Root is permitted |

```yaml
securityContext:
  runAsNonRoot: true
```

> **Important:** This is a critical security setting. Always set to `true` unless root access is strictly required.

---

### `fsGroup`

Specifies the **supplemental group ID** applied to mounted volumes. All files created in mounted volumes will be owned by this group.

```yaml
securityContext:
  fsGroup: 2000
```

For example, if `fsGroup: 2000`, then files created inside the mounted volume will have group ownership `2000`.

---

### `allowPrivilegeEscalation`

Controls whether a container process can gain **more privileges** than its parent process (e.g., via `sudo` or `su`).

| Value | Behavior |
|-------|----------|
| `true` | Privilege escalation allowed |
| `false` | Privilege escalation blocked |

```yaml
securityContext:
  allowPrivilegeEscalation: false
```

> **Best practice:** Always set to `false` to prevent privilege escalation attacks.

---

## Linux Capabilities

Linux capabilities allow fine-grained control over **what privileged operations a container can perform**. They are defined inside the Security Context and apply **only at the container level**.

### Common Capabilities

| Capability | Description |
|------------|-------------|
| `NET_ADMIN` | Manage network interfaces, firewalls, and routing tables |
| `SYS_ADMIN` | Mount filesystems, change kernel parameters, system-level configuration |
| `SYS_TIME` | Change the system clock (used by NTP clients and time sync services) |

---

### `NET_ADMIN`

Grants the ability to:
- Create and configure network interfaces
- Modify firewall rules
- Update routing tables

> **Caution:** Only add this capability if the container explicitly requires network administration. Do not include it by default.

---

### `SYS_ADMIN`

Grants the ability to:
- Mount and unmount filesystems
- Make system-level configuration changes
- Modify kernel parameters

> **Caution:** This is one of the most powerful capabilities. Avoid unless strictly necessary.

---

### `SYS_TIME`

Grants the ability to:
- Modify the system clock
- Used primarily by NTP clients and time synchronization services

---

### `add` and `drop`

Capabilities are managed using `add` and `drop` directives inside the `capabilities` block:

```yaml
securityContext:
  capabilities:
    drop:
      - ALL          # Drop all capabilities first (least-privilege baseline)
    add:
      - NET_ADMIN    # Then explicitly add only what's needed
      - SYS_TIME
```

> **Best practice:** Start by dropping `ALL` capabilities, then explicitly add only what is required.

---

## Example YAML Configurations

### Linux Capabilities (`cap.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: cap-demo
spec:
  containers:
    - name: cap-container
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        capabilities:
          drop:
            - ALL
          add:
            - NET_ADMIN
            - NET_RAW
```

---

### Security Context with User/Group and Volume (`security.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  volumes:
    - name: demo-volume
      emptyDir: {}
  containers:
    - name: secure-container
      image: ubuntu
      command: ["sleep", "3600"]
      volumeMounts:
        - name: demo-volume
          mountPath: /data/demo
```

When you create a file inside `/data/demo`:
- **Owner (user):** UID `1000`
- **Owner (group):** GID `2000` (from `fsGroup`)

---

### Non-Root Enforcement (`root.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nonroot-demo
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
  containers:
    - name: nonroot-container
      image: ubuntu
      command: ["sleep", "3600"]
      securityContext:
        allowPrivilegeEscalation: false
```

---

## Pod Security Standards

Kubernetes provides three built-in **Pod Security Standards** that can be enforced at the namespace level.

| Standard | Security Level | Restriction Level | Description |
|----------|---------------|-------------------|-------------|
| **Privileged** | Low | Low | Unrestricted; allows most capabilities |
| **Baseline** | Medium | Medium | Prevents known privilege escalations |
| **Restricted** | High | High | Enforces current hardening best practices |

### Applying Pod Security Standards to a Namespace

Pod Security Standards are applied via **labels** on the namespace.

**Step 1: Create the namespace**

```bash
kubectl create namespace restricted-ns
```

**Step 2: Apply the security standard label**

```bash
kubectl label namespace restricted-ns \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=latest
```

Available modes:
- `enforce` — Rejects non-compliant pods
- `audit` — Logs violations but allows the pod
- `warn` — Shows warnings but allows the pod

---

### Compliant Pod (Restricted Standard)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  namespace: restricted-ns
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: compliant-container
      image: nginx
      securityContext:
        allowPrivilegeEscalation: false
        capabilities:
          drop:
            - ALL
```

✅ This pod will be **created successfully** — it is fully compliant with the `restricted` standard.

---

### Non-Compliant Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod2
  namespace: restricted-ns
spec:
  containers:
    - name: non-compliant-container
      image: nginx
      securityContext:
        capabilities:
          add:
            - NET_ADMIN
            - SYS_ADMIN
```

❌ This pod will be **rejected** with violations such as:
- Capabilities not dropped
- Prohibited capabilities added (`NET_ADMIN`, `SYS_ADMIN`)
- Missing `runAsNonRoot`
- Missing `allowPrivilegeEscalation: false`
- Missing `seccompProfile`

---

## CKA Exam Tips

> **Sample exam question:** *Create a namespace called `restricted` and configure it with the `restricted` Pod Security Standard.*

Steps to answer:
1. Create the namespace: `kubectl create namespace restricted`
2. Apply the security label using `kubectl label`
3. Create a compliant pod that passes all restricted-level checks (drop ALL capabilities, set `runAsNonRoot: true`, set `allowPrivilegeEscalation: false`)

Key things the `restricted` standard requires:
- `capabilities.drop: [ALL]`
- `allowPrivilegeEscalation: false`
- `runAsNonRoot: true`
- A valid `seccompProfile` (e.g., `RuntimeDefault`)

---

## Quick Reference

```
Security Context
├── runAsUser            → UID to run the container process
├── runAsGroup           → GID to run the container process
├── runAsNonRoot         → Reject pod if it runs as root
├── fsGroup              → Group ownership for mounted volumes
├── allowPrivilegeEscalation → Allow/deny sudo and su
└── capabilities         → (Container-level only)
    ├── add: [CAP_NAME]  → Grant specific capabilities
    └── drop: [ALL]      → Remove all capabilities (then add selectively)

Pod Security Standards (Namespace-level)
├── Privileged   → Low security, low restriction
├── Baseline     → Medium security, medium restriction
└── Restricted   → High security, high restriction
```