# Day 39 — Troubleshooting Worker Node Failures

---

## Scenario 1: Cluster is Broken — Worker Node Not Ready

The master node is running, but one or more worker nodes are in a `NotReady` state.

### Why Nodes Go NotReady

If multiple nodes are unhealthy simultaneously, the most likely cause is a **networking problem** — specifically with the CNI (Container Network Interface) add-on plugin.

- Common plugins: **Calico**, **Weave Net**, **Flannel**
- The plugin may be missing or misconfigured
- This component is responsible for:
  - Pod-to-pod network communication
  - Node-to-node networking

### Check if a Network Plugin is Installed

```bash
kubectl get pods -A
kubectl get pods -n <namespace>
```

> The 3 default namespaces will always be present. Additional namespaces (e.g. `calico-system`) indicate a CNI plugin is installed.

### Inspect the CNI Config

```bash
cd /etc/cni/net.d
```

Two files should exist under `net.d`:

- `10-<plugin>.conf` or `10-<plugin>.conflist`
- `<plugin>-kubeconfig`

If these are absent or malformed, the CNI plugin is not set up correctly.

---

## Step-by-Step: Fixing Worker Nodes

### Worker Node 1 — kubelet Inactive

SSH into the worker node, then:

**1. Check kubelet status**

```bash
service kubelet status
```

If it shows **inactive (dead)**, the service has stopped.

**2. Start the service and verify**

```bash
sudo service kubelet start
sudo service kubelet status
```

> Press `Shift + G` in the status pager to jump to the most recent log lines.

---

### Worker Node 2 — kubelet Stuck Activating

SSH into node 2. If `service kubelet status` shows **activating** (not active), the service is failing to start due to a configuration error.

**1. Read the kubelet logs with `journalctl`**

```bash
journalctl -u kubelet
```

Press `Shift + G` to jump to the latest entries.

**2. Understand log prefixes**

| Prefix | Meaning |
|--------|---------|
| `I0712` | Info — safe to skip |
| `E0712` | Error — investigate these |

Look for an `E` line pointing to a bad path in the kubelet config.

**3. Open the kubelet config file**

```bash
cd /var/lib/kubelet
vi config.yaml
```

**4. Fix the `clientCAFile` path**

```yaml
# Wrong (typo in "kubernetes")
clientCAFile: /etc/kubenetes/pki/ca.cert

# Correct
clientCAFile: /etc/kubernetes/pki/ca.crt
```

Verify the file exists at that path before saving:

```bash
ls /etc/kubernetes/pki/ca.crt
```

**5. Restart and confirm**

```bash
sudo service kubelet restart
sudo service kubelet status
```

The node should now show **active (running)** and rejoin the cluster.

---

## Verify Both Nodes are Healthy

From the master node:

```bash
kubectl get nodes
```

All nodes should report `Ready`.