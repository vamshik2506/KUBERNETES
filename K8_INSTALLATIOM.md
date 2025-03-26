# **Installation of Kubernetes Cluster**

## **Run These Commands in Both Master and Worker Nodes**

### **Step 1: Update and Install Required Packages**
```bash
sudo apt-get update
sudo apt install apt-transport-https curl -y
```

### **Step 2: Install Container Runtime (Containerd)**
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install containerd.io -y
```

### **Step 3: Configure Containerd**
```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i -e 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
```

### **Step 4: Add Kubernetes Repository**
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```

### **Step 5: Install Kubernetes Components**
```bash
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### **Step 6: Enable and Start Kubernetes Services**
```bash
sudo systemctl enable --now kubelet
```

### **Step 7: Disable Swap and Enable Network Settings**
```bash
sudo swapoff -a
sudo modprobe br_netfilter
sudo sysctl -w net.ipv4.ip_forward=1
```

---

## **Run These Commands Only in Master Node**

### **Step 1: Initialize the Kubernetes Cluster**
```bash
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```

### **Step 2: Configure kubectl for the Master Node**
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### **Step 3: Install Network Plugin (Flannel)**
```bash
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

---

## **Run These Commands in Worker Nodes**

### **Step 1: Perform Pre-Flight Checks**
Before joining the cluster, reset the worker node to ensure a clean setup:
```bash
sudo kubeadm reset pre-flight checks
```

### **Step 2: Join the Worker Node to the Cluster**
🔹 **Switch to the root user first to avoid permission issues:**
```bash
sudo su
```

🔹 **Now, paste the join command from the master node and append `--v=5` for detailed logs:**
```bash
<your-token --v=5>
```
📌 **Replace `<your-token>` with the actual join command from your master node.**

---

## **Verify the Worker Nodes (Run on Master Node)**
After all worker nodes have joined the cluster, verify their status from the **master node**:
```bash
kubectl get nodes
```

If everything is set up correctly, the worker nodes should appear in the **Ready** state.

---

### 🎉 **Kubernetes Cluster Setup is Complete!**
Your Kubernetes cluster is now successfully set up with a master node and worker nodes. 🚀

