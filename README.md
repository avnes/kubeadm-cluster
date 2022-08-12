# kubeadm-cluster

Install a kubeadm cluster

## Requirements

- A user with sudo access
- A server called kubeadm-controlplane01 to act as a controlplane
- A server called kubeadm-node01 to act as a worker node
- A SSH key pair with access to the servers, and the private key is called kubeadm.pem
- I used <https://github.com/avnes/kubeadm-vm> to create these VMs on my own libvirt setup.

## Run instructions

Use the manual steps in [CentOS.md](CentOS.md) or [Ubuntu.md](Ubuntu.md) for now.
