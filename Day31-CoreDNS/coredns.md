Here’s your **same notes, just cleaned + structured + easy wording** (no heavy terms, no over-complication). I kept your flow intact and improved clarity.

---

# 🔹 Kubernetes CoreDNS – Clean Notes

## 🔸 Problem Scenario (DNS Issue)

* Currently, **2 pods are running**
* Trying to `curl nginx1` (service name)

👉 Issue:

* Request is **failing / terminating**
* Because:

  * `nginx1` is a **service name (hostname)**
  * DNS is **not resolving it to an IP**

👉 Observation:

* Using **Service Name → ❌ Not working**
* Using **Service IP → ✅ Working**

👉 Conclusion:

* DNS is **misconfigured / not working properly**
* Service name is not resolving to IP

---

## 🔸 Fix: Check CoreDNS

👉 Check CoreDNS pods:

```
kubectl get pods -n kube-system
```

* CoreDNS was not working properly

👉 Fix:

* Scaled CoreDNS to **2 pods**

👉 After fix:

* CoreDNS becomes **healthy**
* Now:

```
curl nginx1
```

✅ Working

---

## 🔸 What is CoreDNS?

* CoreDNS is a **DNS Server in Kubernetes**
* It converts:

  * **Service Name → IP Address**
* It acts like:

  * A **central DNS system inside the cluster**

👉 Example:

```
nginx1 → 10.x.x.x
```

---

## 🔸 Kube-DNS Service

* CoreDNS runs behind a service called:

  * **kube-dns**
* This service:

  * Works on **port 53 (DNS port)**

---

## 🔸 Inside a Pod (Important)

👉 Check file:

```
/etc/resolv.conf
```

What you will see:

* **nameserver → kube-dns service IP**
* This is auto-added when pod is created

👉 Meaning:

* Every pod automatically knows:

  * Which DNS server to use

---

### 🔸 Search Domains

Inside `/etc/resolv.conf`:

* You will see **search entries**

👉 Purpose:

* Allows short names like:

```
nginx1
```

instead of:

```
nginx1.default.svc.cluster.local
```

---

## 🔸 /etc/hosts File

* Used to manually map:

  * IP → Hostname

👉 But in Kubernetes:

* ❌ We don’t need to use it manually
* ✅ CoreDNS handles everything automatically

---

## 🔸 CoreDNS Important Config

👉 Command:

```
kubectl describe cm coredns -n kube-system
```

👉 This is a **ConfigMap mounted into CoreDNS pod**

---

### 🔹 What’s inside CoreDNS config?

* Health check:

  * Runs every **5 seconds**
* Metrics:

  * Exposed on **port 9153 (Prometheus)**
* Forwarding:

  * Requests go to `/etc/resolv.conf`
* Load balancing:

  * Uses **round-robin**
* Auto reload:

  * Updates when config changes

---

## 🔸 How CoreDNS Works (Simple Flow)

1. Pod sends request → `nginx1`
2. Request goes to → **CoreDNS**
3. CoreDNS checks config
4. Returns → **Service IP**
5. Pod connects to that IP

---

## 🔸 Troubleshooting CoreDNS

### ❗ Issue: CoreDNS not working

Even if pod is running:

* Still DNS may fail

---

### 🔹 Common Reasons

#### 1. Network Plugin Issue

* CoreDNS depends on **CNI (network plugin)**

👉 Example:

* Calico

If Calico is not working:

* CoreDNS will also fail

---

#### 2. Check All Pods

```
kubectl get pods -A
```

👉 Ensure:

* CoreDNS → Running
* CNI pods → Running

---

#### 3. Firewall Issue

* Firewall should be **disabled on all nodes**

---

#### 4. Calico Issue

If Calico node not running:

* Enable **IP auto-detection**

---

## 🔸 Practice (Must Do)

👉 Official debugging guide:

*

---

## 🔥 Final Quick Summary

* CoreDNS = **DNS server in K8s**
* Converts:

  * Service Name → IP
* kube-dns = **Service exposing CoreDNS**
* `/etc/resolv.conf` = DNS config inside pod
* No need to use `/etc/hosts`
* If DNS fails:

  * Check CoreDNS
  * Check CNI (Calico)
  * Check firewall

---