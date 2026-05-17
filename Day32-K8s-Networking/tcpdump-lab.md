Perfect—this is where your understanding goes from “I know theory” → “I can debug production issues.”
We’ll do a **real tcpdump lab inside Kubernetes**, step-by-step, exactly how you’d do in a real cluster.

---

# 🧪 Lab Goal

You will **see the packet flow live**:

* Pod A → Service → Pod B
* Observe:

  * DNS (CoreDNS)
  * kube-proxy (DNAT)
  * CNI routing
  * Actual packet movement

---

# 🏗️ Step 0: Setup Environment

Create a namespace:

```bash
kubectl create ns net-lab
```

---

## Deploy backend pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: backend
  namespace: net-lab
  labels:
    app: backend
spec:
  containers:
  - name: nginx
    image: nginx
```

```bash
kubectl apply -f backend.yaml
```

---

## Expose as Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-svc
  namespace: net-lab
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 80
```

```bash
kubectl apply -f svc.yaml
```

---

## Deploy client pod (with tools)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: client
  namespace: net-lab
spec:
  containers:
  - name: client
    image: nicolaka/netshoot
    command: ["sleep", "3600"]
```

```bash
kubectl apply -f client.yaml
```

---

# 🔍 Step 1: Get Pod & Node Info

```bash
kubectl get pods -o wide -n net-lab
```

Note:

* Pod IPs
* Node where pods are running

---

# 🌐 Step 2: Verify DNS (CoreDNS in action)

Inside client pod:

```bash
kubectl exec -it client -n net-lab -- bash
```

Run:

```bash
nslookup backend-svc
```

👉 This hits **CoreDNS**

Expected:

```id="dnsout"
Name: backend-svc
Address: 10.96.x.x
```

---

# 📡 Step 3: Start tcpdump (Packet Capture)

Still inside client pod:

```bash
tcpdump -i any host backend-svc -nn
```

👉 This will capture:

* DNS packets
* HTTP packets

---

# 🚀 Step 4: Generate Traffic

In another terminal:

```bash
kubectl exec -it client -n net-lab -- curl backend-svc
```

---

# 🔥 What You Will See in tcpdump

---

## 1️⃣ DNS Query

```id="dnsdump"
IP client → DNS: query backend-svc
IP DNS → client: response 10.96.x.x
```

✔️ Confirms CoreDNS working

---

## 2️⃣ Request to Service IP

```id="svcflow"
SRC: client-pod-ip
DEST: service-cluster-ip
```

But wait…

👉 No pod has this IP → virtual!

---

# ⚙️ Step 5: Observe kube-proxy (DNAT)

Now go to **node where client pod is running**

SSH into node:

```bash
sudo tcpdump -i any port 80 -nn
```

You’ll see:

```id="dnatflow"
DEST: backend pod IP (NOT service IP)
```

👉 That means:

Handled by **kube-proxy**

✔️ kube-proxy converted:

```
Service IP → Pod IP
```

---

# 🔌 Step 6: Observe CNI Routing

Now check if pods are on:

---

## Same Node

You’ll see:

```id="samenode"
client → backend (direct)
```

✔️ No encapsulation

---

## Different Node (IMPORTANT)

Run tcpdump on both nodes:

```bash
tcpdump -i any host <backend-pod-ip>
```

You may see:

```id="overlay"
VXLAN / encapsulated packet
```

👉 This is done by:

**Container Network Interface**

✔️ CNI handles:

* Routing
* Encapsulation
* Delivery

---

# 🔥 Step 7: Observe NAT

Look carefully:

---

## DNAT (always for Service)

```id="dnatexample"
10.96.x.x → 10.244.x.x
```

---

## SNAT (sometimes)

If cross-node:

```id="snatexample"
SRC: pod IP → node IP
```

✔️ Ensures return traffic works

---

# 🧭 Full Observed Flow (What you just saw)

```id="fulltrace"
1. DNS request → CoreDNS
2. Service IP returned
3. Packet sent to ClusterIP
4. kube-proxy → DNAT to Pod IP
5. CNI → routes packet
6. Packet reaches backend pod
7. Response returns
```

---

# 🧠 Advanced Debug (Interview Killer)

Run this on node:

```bash
iptables -t nat -L -n | grep <service-ip>
```

You’ll see rules created by kube-proxy.

---

# 🎯 Real Interview Answer (After this lab)

If they ask:

👉 “How do you debug networking issue?”

You say:

> I use tcpdump inside pod and node to trace packet flow. First verify DNS via CoreDNS, then check Service IP translation using kube-proxy rules, and finally validate CNI routing and NAT behavior across nodes.

---

# 💥 Next Level (Highly Recommended)

If you want, I’ll push you harder:

* ❌ Break DNS → debug CoreDNS
* ❌ Break Service → debug kube-proxy
* ❌ Break CNI → debug pod communication

Just say: **“give me failure scenarios”**
