# kubeadm on CentOS Stream

- Tested with success on CentOS Stream 9
- Failed on CentOS Stream 8

```bash
sudo dnf install -y yum-utils


sudo modprobe overlay
sudo modprobe br_netfilter

cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sudo sysctl --system

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

# On CentOS Stream 8: dnf remove -y podman

sudo dnf install -y containerd.io

sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
sudo systemctl restart containerd

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF

sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

sudo systemctl enable --now kubelet

sudo swapoff -a

VERSION="v1.24.1"
curl -L https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-${VERSION}-linux-amd64.tar.gz --output crictl-${VERSION}-linux-amd64.tar.gz
sudo tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
rm -f crictl-$VERSION-linux-amd64.tar.gz


# On controlplane:
sudo kubeadm init --pod-network-cidr 10.244.0.0/16 --apiserver-advertise-address=192.168.122.29 --ignore-preflight-errors 'NumCPU'

mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

sudo sh -c "KUBECONFIG=/etc/kubernetes/admin.conf kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=v1.24.3"
sudo sh -c "KUBECONFIG=/etc/kubernetes/admin.conf kubectl get nodes -A -o wide"

# On worker node:

sudo kubeadm join 192.168.122.29:6443 --token b19ubu.h7ultfscvupdu2j5 \
        --discovery-token-ca-cert-hash sha256:649187d6f0680d3b464bd62e8195807446abfad2aeff673be6ad86348866cbdc

```

## Remove kubeadm

```bash

# On worker node:

sudo kubeadm reset


# On controlplane:

sudo kubeadm reset
```
