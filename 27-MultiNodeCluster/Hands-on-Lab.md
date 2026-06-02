## Hands-on Guide: Multi-Node Kubernetes Cluster v1.29 using `kubeadm` on Virtual Machines

We’ll build:

* **1 Control Plane (Master)**
* **2 Worker Nodes**
* Kubernetes **v1.29**
* Networking with **Calico**
* All nodes in **Ready** state

This is excellent real-world DevOps / CKA practice.

---

# Lab Topology

| Node Role | Hostname    | IP           |
| --------- | ----------- | ------------ |
| Master    | k8s-master  | 192.168.1.10 |
| Worker1   | k8s-worker1 | 192.168.1.11 |
| Worker2   | k8s-worker2 | 192.168.1.12 |

Use Ubuntu 22.04 VMs (recommended).

Minimum per VM:

* 2 CPU
* 2 GB RAM (worker)
* 4 GB RAM (master)

---

# Step 1: Prepare All VMs

Run on **all 3 nodes**

## Set hostname

### Master

```bash
sudo hostnamectl set-hostname k8s-master
```

### Worker1

```bash
sudo hostnamectl set-hostname k8s-worker1
```

### Worker2

```bash
sudo hostnamectl set-hostname k8s-worker2
```

---

## Update Hosts File

Run on all nodes:

```bash
sudo nano /etc/hosts
```

Add:

```bash
192.168.1.10 k8s-master
192.168.1.11 k8s-worker1
192.168.1.12 k8s-worker2
```

---

# Step 2: Disable Swap

Kubernetes requires swap OFF.

Run on all nodes:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

Check:

```bash
free -h
```

---

# Step 3: Enable Kernel Modules

Run on all nodes:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

---

# Step 4: Set Sysctl Params

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-ip6tables=1
EOF
```

Apply:

```bash
sudo sysctl --system
```

---

# Step 5: Install Container Runtime (containerd)

Run on all nodes:

```bash
sudo apt update
sudo apt install -y containerd
```

Generate config:

```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

Enable systemd cgroup:

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

Restart:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Check:

```bash
systemctl status containerd
```

---

# Step 6: Install Kubernetes 1.29 Packages

Run on all nodes.

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg
```

Add repo key:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | \
sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add repo:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' |
sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Install:

```bash
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
```

Hold versions:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

Check versions:

```bash
kubeadm version
kubectl version --client
```

---

# Step 7: Initialize Master Node

Run **only on master**

```bash
sudo kubeadm init \
--apiserver-advertise-address=192.168.1.10 \
--pod-network-cidr=192.168.0.0/16
```

After success, copy kubeconfig:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

---

# Step 8: Install Calico Network Plugin

Run on master:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml
```

Check:

```bash
kubectl get pods -n kube-system
```

Wait until Calico pods are Running.

---

# Step 9: Join Worker Nodes

On master, get join command:

```bash
kubeadm token create --print-join-command
```

Example:

```bash
sudo kubeadm join 192.168.1.10:6443 \
--token abcdef.123456789 \
--discovery-token-ca-cert-hash sha256:xxxx
```

Run that command on:

* worker1
* worker2

---

# Step 10: Verify Cluster

Run on master:

```bash
kubectl get nodes
```

Expected:

```bash
NAME           STATUS   ROLES           AGE   VERSION
k8s-master     Ready    control-plane   5m    v1.29.x
k8s-worker1    Ready    <none>          2m    v1.29.x
k8s-worker2    Ready    <none>          2m    v1.29.x
```

---

# Step 11: Test Workload

```bash
kubectl create deployment nginx --image=nginx
kubectl get pods -o wide
```

You should see pods scheduled on workers.

---

# Troubleshooting

## If node NotReady:

Check kubelet:

```bash
systemctl status kubelet
```

Check containerd:

```bash
systemctl status containerd
```

Check Calico:

```bash
kubectl get pods -n kube-system
```

---

# Real Interview Explanation

> We use kubeadm for production-style cluster bootstrap.
> containerd as runtime.
> kubelet runs node agent.
> kubeadm initializes control plane and joins workers.
> kubectl is CLI.
> Calico provides Pod networking + Network Policies.

---

# CKA Exam Fast Revision Commands

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
kubectl apply -f calico.yaml
kubeadm token create --print-join-command
kubectl get nodes
```

---

