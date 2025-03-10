# kubernetes_lab
practicing kubernetes

Step 1: Prepare the System
1.1 Update the System

sudo apt update && sudo apt upgrade -y
1.2 Disable Swap (Required for Kubernetes)

sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab

1.3 Load Required Kernel Modules

cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
sudo modprobe overlay
sudo modprobe br_netfilter

1.4 Set Kernel Parameters

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
sudo sysctl --system

Step 2: Install Kubernetes Components
2.1 Install Container Runtime (containerd)

sudo apt install -y apt-transport-https ca-certificates curl
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y containerd.io

2.2 Configure containerd

sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
sudo systemctl enable containerd

2.3 Install Kubernetes Tools (kubeadm, kubelet, kubectl)

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

Step 3: Initialize the Kubernetes Cluster
Run this command on the master node only
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
If the command is successful, it will output a kubeadm join command that you can use to join worker nodes to the cluster.

3.1 Set Up Kubeconfig for kubectl

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
3.2 Verify the cluster status:

kubectl get nodes
Step 4: Install a Pod Network Add-on
A pod network is required for communication between Kubernetes pods.

4.1 Install Calico CNI (Recommended)

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
OR

4.2 Install Flannel CNI

kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

Step 5: Join Worker Nodes to the Cluster
Run this command on each worker node Use the kubeadm join command that was generated in Step 3. If you lost it, retrieve it with:

kubeadm token create --print-join-command
Then run the command on worker nodes, for example:

sudo kubeadm join 192.168.0.114:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
After joining, verify the nodes from the master:


kubectl get nodes
It should show all nodes as Ready.

Step 6: Deploy a Sample Application (Optional)
To test your cluster, deploy a simple Nginx application:

kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get svc nginx
Find the NodePort and access Nginx in a browser:

http://<worker-node-ip>:<NodePort>

Step 7: Enable Kubernetes Dashboard (Optional)

kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl proxy
Access the dashboard at:

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
Next Steps

🎯 Congratulations! Your Kubernetes cluster is now running! 🎯

From here, you can:

Deploy and manage applications.
Learn about Helm charts for package management.
Implement monitoring tools like Prometheus & Grafana.
Configure Ingress for better networking.
