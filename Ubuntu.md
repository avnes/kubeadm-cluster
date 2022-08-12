# Test kubeadm on Ubuntu Server


```bash
sudo apt update
sudo apt install -y openssh-server

sudo systemctl enable sshd
sudo systemctl start sshd

sudo apt install -y containerd apt-transport-https ca-certificates curl

sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo mkdir -p /etc/containerd
sudo sh -c "containerd config default > /etc/containerd/config.toml"
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd

sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
sudo echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt update
sudo apt install -y kubelet kubeadm kubectl

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system

sudo swapoff -a


# On controlplane:
sudo kubeadm init --pod-network-cidr 10.244.0.0/16 --ignore-preflight-errors 'NumCPU'

mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

sudo sh -c "KUBECONFIG=/etc/kubernetes/admin.conf kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=v1.24.3"
sudo sh -c "KUBECONFIG=/etc/kubernetes/admin.conf kubectl get nodes -A -o wide"

Copy the "kubeadm join" instructions as they will be used on the worker node.

# On worker node:

sudo kubeadm join ......<instructions>

```

## Remove kubeadm

```bash

# On worker node:

sudo kubeadm reset


# On controlplane:

sudo kubeadm reset
```
