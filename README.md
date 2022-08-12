# kubeadm-cluster

Install a kubeadm cluster

## Requirements

- A user with sudo access
- A server called kubeadm-ctrl01 to act as a controlplane (or edit ansible/inventory/hosts.yaml to use another name)
- A server called kubeadm-node01 to act as a worker node (or edit ansible/inventory/hosts.yaml to use another name)
- Both servers configured with an ansible user that has password-less sudo access
- A SSH key pair with access to the servers, and the private key is called kubeadm.pem (or update bin/01_run_ansible.sh with other key name)

## Run instructions

WIP: This is still work in progress. Use the manual steps in [CentOS.md](CentOS.md) or [Ubuntu.md](Ubuntu.md) for now.

```bash
bin/01_run_ansible.sh
```
