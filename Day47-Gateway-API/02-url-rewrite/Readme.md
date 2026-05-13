# What This Route Does

```text id="jlwm67"
Client Request:
/get

Gateway rewrites it to:
/replace
```

before forwarding to backend service.

---

# Request Flow

```text id="5xcyhb"
Client
  ↓
/get
  ↓
Gateway URLRewrite filter
  ↓
/replace
  ↓
backend service
  ↓
backend pod
```

---

# Test Command

```bash id="jlwm98"
curl -H "Host: path.rewrite.example" http://localhost:8080/get
```

---

# Important Fixes Done

## 1. Added missing path type

You missed:

```yaml id="jlwm11"
type: PathPrefix
```

---

## 2. Fixed indentation

Wrong:

```yaml id="jlwm12"
filters:
  backendRefs:
```

Correct:

```yaml id="jlwm13"
filters:
backendRefs:
```

Both are sibling fields under `rules`.

---

# Important Gateway API Concept

Filters are used for:

* URL rewrite
* Header modification
* Redirects
* Request mirroring
* Authentication (controller-dependent)

Very important interview topic.

---

# URLRewrite Types

## ReplacePrefixMatch

```text id="jlwm14"
/get/test
↓
/replace/test
```

---

## ReplaceFullPath

Completely replaces URL path.

Example:

```yaml id="jlwm15"
type: ReplaceFullPath
replaceFullPath: /newpath
```

---

# Debugging Commands

```bash id="jlwm16"
kubectl describe httproute http-filter-url-rewrite
```

Look for:

```text id="jlwm17"
Accepted=True
ResolvedRefs=True
```

---

# Expected Echo Output

You should see request reaching backend as:

```text id="jlwm18"
GET /replace HTTP/1.1
```

even though client requested:

```text id="jlwm19"
/get
```
