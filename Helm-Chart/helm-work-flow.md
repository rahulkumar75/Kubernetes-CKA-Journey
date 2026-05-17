# Real Workflow Example

## 1. Initial Install

```bash id="w6fr8g"
helm install payments-release my-org/payments
```

Helm creates:

```text id="xzjlwm"
Revision 1
```

---

## 2. Upgrade App

Maybe you changed:

* image version
* replicas
* env vars
* resources

Then:

```bash id="i1hnd7"
helm upgrade payments-release my-org/payments
```

Now:

```text id="wq7lpn"
Revision 2
```

---

## 3. Something Breaks ❌

Pods crash:

```bash id="2w8m91"
kubectl get pods
```

You see:

```text id="dzgpcn"
CrashLoopBackOff
```

---

# 4. Check Release History

```bash id="t7t7lr"
helm history payments-release
```

Example output:

```text id="mq5gfm"
REVISION  STATUS      DESCRIPTION
1         deployed    Install complete
2         deployed    Upgrade complete
```

---

# 5. Roll Back


```bash id="9nkqgh"
helm rollback <release-name> <revision-number>
```

Example:

```bash id="4u3v8u"
helm rollback payments-release 1
```

Meaning:

```text id="sybjlwm"
Restore payments-release to revision 1
```

Now Helm restores:

* old Deployment
* old ConfigMaps
* old values
* old image tag
* old manifests

Cluster returns to stable state.
---

`helm rollback` is used to restore a previous working version of your application deployment.

If a new Helm upgrade breaks your app, you can quickly go back to an older stable release.

---

# Think Like This

```text id="1z9tmn"
git revert   -> source code rollback
helm rollback -> Kubernetes deployment rollback
```

---

# Important Interview Point

Helm stores release history inside Kubernetes secrets/configmaps.

Check:

```bash id="fy8bx5"
kubectl get secrets
```

You may see:

```text id="e9w0mz"
sh.helm.release.v1.payments-release.v1
sh.helm.release.v1.payments-release.v2
```

These store revision history.

---

# Common Real-World Usage

Production deployment failed after:

```bash id="wo3e9w"
helm upgrade
```

Immediate recovery:

```bash id="tovp93"
helm rollback payments-release <stable-revision>
```

This is one of Helm’s biggest advantages in DevOps CI/CD pipelines.
