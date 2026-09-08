# Cheat Sheet

```text
ConfigMap
   │
   ├── envFrom
   │      └── ALL keys
   │
   ├── configMapKeyRef
   │      └── ONE key
   │
   └── Volume
          └── keys become files
```

```text
Secret
   │
   ├── secretKeyRef
   │      └── ONE key
   │
   ├── envFrom
   │      └── ALL keys
   │
   └── Volume
          └── keys become files
```

### Must Remember

* **ConfigMap = non-sensitive configuration**
* **Secret = sensitive data**
* **`envFrom`**** = all keys**
* **`*KeyRef`**** = one key**
* **Volume = files**
* **Pod + ConfigMap/Secret = same namespace**
* **Env injection does not automatically update**
* **Normal volume mounts update eventually**
* **`subPath`**** mounts do not auto-update**
* **Base64 ≠ encryption**
* **Immutable ConfigMap → delete and recreate**