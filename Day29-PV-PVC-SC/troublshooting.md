# 🚨 Troubleshooting (Exam Style Mistakes)

## ❌ PVC Pending

Check:

```bash id="jlwmt1"
kubectl describe pvc cka-pvc
```

Reasons:

* `storageClassName` mismatch
* PV size smaller than PVC
* accessModes mismatch

---

## ❌ Pod Pending

Reason:

PVC not Bound.

Check:

```bash id="jlwmt2"
kubectl get pvc
```

---

## ❌ Mistake Example 1

Wrong:

```yaml id="jlwmt3"
accessMode:
```

Correct:

```yaml id="jlwmt4"
accessModes:
```

---

## ❌ Mistake Example 2

Wrong:

```yaml id="jlwmt5"
volume:
```

Correct:

```yaml id="jlwmt6"
volumes:
```

---

# ⚡ 2-Minute CKA Shortcut

```bash id="jlwmt7"
kubectl get pv
kubectl get pvc
kubectl describe pvc
kubectl edit pod web-pod
```

---
