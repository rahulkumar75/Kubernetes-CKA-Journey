# 🧠 Cheat Sheet

```text
Liveness
→ Is the container healthy?
→ Failure → Restart

Readiness
→ Can the Pod receive traffic?
→ Failure → Remove from Service endpoints

Startup
→ Has the application finished starting?
→ Protects slow-starting applications
```

### Probe mechanisms

```text
HTTP GET
→ Check HTTP endpoint

TCP Socket
→ Check TCP connection

Exec
→ Run command inside container
→ Exit 0 = Success
```

---