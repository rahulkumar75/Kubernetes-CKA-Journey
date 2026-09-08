
# Cheat Sheet

```text
Authentication
    ↓
Who are you?

Authorization
    ↓
What can you do?

RBAC
    ↓
Role + RoleBinding

Role
    ↓
Defines permissions

RoleBinding
    ↓
Assigns Role to User / Group / ServiceAccount

get/list/watch
    ↓
Read

create/update/patch/delete
    ↓
Write/manage
```

### Must-remember commands

```bash
kubectl auth can-i get pods --as=adam
kubectl auth can-i --list --as=adam

kubectl get roles -A
kubectl get rolebindings -A

kubectl get clusterroles
kubectl get clusterrolebindings

kubectl config get-contexts
kubectl config current-context
kubectl config use-context adam
```

### Final Memory Rule

> **Authentication → Who are you?**
> **RBAC → What can you do?**
> **Role → What permissions?**
> **RoleBinding → Who gets them?**
