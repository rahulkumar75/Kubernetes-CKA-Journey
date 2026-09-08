# 🔧 Troubleshooting Probes

When a Pod has probe problems:

### Step 1 — Check Pod

```bash
kubectl get pod <pod-name>
```

### Step 2 — Describe Pod

```bash
kubectl describe pod <pod-name>
```

Check:

```text
Events
Liveness probe failed
Readiness probe failed
Startup probe failed
```

### Step 3 — Check logs

```bash
kubectl logs <pod-name>
```

For a previous container instance:

```bash
kubectl logs <pod-name> --previous
```

### Step 4 — Verify the endpoint/port

For HTTP:

```bash
kubectl exec -it <pod-name> -- curl -I http://localhost:80/
```

For TCP, verify that the application is actually listening on the configured port.

For Exec:

```bash
kubectl exec -it <pod-name> -- cat /tmp/healthy
```

---