# Creating a Kubernetes Cluster with kubeadm

Kubernetes is the de facto standard for container orchestration. While managed services such as Amazon EKS, Google GKE, and Azure AKS abstract away the control plane, understanding how Kubernetes works under the hood is an essential skill for any DevOps or Platform Engineer.

In this guide, we'll build a Kubernetes cluster from scratch using **kubeadm** on two Ubuntu virtual machines running in Vagrant:

* **Control Plane** (Master Node)
* **Worker Node**

The goal is to understand the Kubernetes bootstrap process and manually create a cluster without automation tools.

---

## Architecture

Our cluster consists of two virtual machines:

```text
+----------------------+
|    Control Plane     |
|----------------------|
| kube-apiserver       |
| etcd                 |
| kube-controller      |
| kube-scheduler       |
+----------+-----------+
           |
           |
           v
+----------------------+
|      Worker Node     |
|----------------------|
| kubelet              |
| kube-proxy           |
| containerd           |
+----------------------+
```

Both nodes communicate through a private network configured by Vagrant.

---

## Prerequisites

Before starting, ensure you have:

* VirtualBox installed
* Vagrant installed
* Ubuntu 24.04 virtual machines
* At least:

  * 2 CPUs
  * 2 GB RAM per VM

---

## Vagrantfile

Create the following `Vagrantfile`:

```ruby
Vagrant.configure("2") do |config|

  config.vm.define "control-plane" do |cp|
    cp.vm.box = "ubuntu/noble64"
    cp.vm.hostname = "control-plane"
    cp.vm.network "private_network", ip: "192.168.56.10"

    cp.vm.provider "virtualbox" do |vb|
      vb.memory = 4096
      vb.cpus = 2
    end
  end

  config.vm.define "worker" do |worker|
    worker.vm.box = "ubuntu/noble64"
    worker.vm.hostname = "worker"
    worker.vm.network "private_network", ip: "192.168.56.11"

    worker.vm.provider "virtualbox" do |vb|
      vb.memory = 2048
      vb.cpus = 2
    end
  end

end
```

Start the virtual machines:

```bash
vagrant up
```

Connect to the control plane:

```bash
vagrant ssh control-plane
```

Connect to the worker:

```bash
vagrant ssh worker
```

---

# Step 1: Configure Hostnames

Control Plane:

```bash
sudo hostnamectl set-hostname control-plane
```

Worker:

```bash
sudo hostnamectl set-hostname worker
```

Add both nodes to `/etc/hosts`:

```bash
192.168.56.10 control-plane
192.168.56.11 worker
```

---

# Step 2: Disable Swap

Kubernetes requires swap to be disabled.

Run on both nodes:

```bash
sudo swapoff -a
```

Disable swap permanently:

```bash
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

Verify:

```bash
free -h
```

Swap should show:

```text
Swap: 0B
```

---

# Step 3: Configure Kernel Modules

Run on both nodes:

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

Load modules:

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

Configure networking:

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF
```

Apply changes:

```bash
sudo sysctl --system
```

---

# Step 4: Install Containerd

Update packages:

```bash
sudo apt update
```

Install containerd:

```bash
sudo apt install -y containerd
```

Generate default configuration:

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

Enable Systemd cgroups:

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' \
/etc/containerd/config.toml
```

Restart containerd:

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

Verify:

```bash
systemctl status containerd
```

---

# Step 5: Install Kubernetes Components

Run on both nodes.

Install dependencies:

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg
```

Add Kubernetes repository:

```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```

Install packages:

```bash
sudo apt update

sudo apt install -y \
kubelet \
kubeadm \
kubectl
```

Prevent automatic upgrades:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

Verify:

```bash
kubeadm version
kubectl version --client
```

---

# Step 6: Initialize the Control Plane

Run only on the control-plane node:

```bash
sudo kubeadm init \
  --apiserver-advertise-address=192.168.56.10 \
  --pod-network-cidr=10.244.0.0/16
```

The initialization process:

* Creates certificates
* Starts etcd
* Starts the API Server
* Starts the Controller Manager
* Starts the Scheduler
* Generates kubeconfig files
* Generates the worker join token

kubeadm will display a command similar to:

```bash
kubeadm join 192.168.56.10:6443 \
--token xxxxxx.xxxxxxxxxxxxxxxx \
--discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxx
```

Save this command.

---

# Step 7: Configure kubectl

Run on the control plane:

```bash
mkdir -p $HOME/.kube

sudo cp /etc/kubernetes/admin.conf \
$HOME/.kube/config

sudo chown $(id -u):$(id -g) \
$HOME/.kube/config
```

Verify cluster access:

```bash
kubectl get nodes
```

At this stage:

```text
control-plane   NotReady
```

This is expected because networking is not installed yet.

---

# Step 8: Install Flannel CNI

A Kubernetes cluster requires a Container Network Interface (CNI) so Pods can communicate with each other.

Install Flannel:

```bash
kubectl apply -f \
https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

Verify:

```bash
kubectl get pods -n kube-flannel
```

Wait until all Flannel pods are running.

---

# Step 9: Join the Worker Node

Run the command generated by `kubeadm init` on the worker node:

```bash
sudo kubeadm join 192.168.56.10:6443 \
--token xxxxxx.xxxxxxxxxxxxxxxx \
--discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxx
```

This command:

1. Discovers the control plane
2. Validates the cluster certificate
3. Registers the node
4. Starts kubelet
5. Joins the cluster securely

---

# Step 10: Verify the Cluster

Back on the control plane:

```bash
kubectl get nodes
```

Expected output:

```text
NAME            STATUS   ROLES           AGE
control-plane   Ready    control-plane   5m
worker          Ready    <none>          2m
```

Your cluster is now operational.

---

# Deploying NGINX

Let's deploy a simple application.

Create a Deployment with three replicas:

```bash
kubectl create deployment nginx \
  --image=nginx:1.27 \
  --replicas=3
```

Verify:

```bash
kubectl get deployments
```

```bash
kubectl get pods
```

Example output:

```text
NAME                     READY
nginx-xxxxx              1/1
nginx-yyyyy              1/1
nginx-zzzzz              1/1
```

Expose the Deployment:

```bash
kubectl expose deployment nginx \
  --type=NodePort \
  --port=80
```

Check the Service:

```bash
kubectl get svc
```

Example:

```text
NAME    TYPE       PORT(S)
nginx   NodePort   80:30080/TCP
```

You can now access NGINX using:

```text
http://<worker-ip>:30080
```

or

```text
http://<control-plane-ip>:30080
```

---

# Conclusion

In this tutorial, we created a Kubernetes cluster manually using kubeadm and two Ubuntu virtual machines. We installed containerd, kubelet, kubeadm, and kubectl, initialized the control plane, joined a worker node, configured Flannel networking, and deployed an NGINX application with three replicas.

Understanding the manual kubeadm workflow provides valuable insight into how Kubernetes clusters are built and operated behind managed services such as EKS, GKE, and AKS.

