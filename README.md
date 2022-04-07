#  Kubernetes Cluster Using Kubeadm on Ubuntu 20.04


## Prerequisites

- SSH accces configuration for the machines
- 3 servers running Ubuntu 20.04 with at least 2GB RAM and 2 vCPUs.  
- Ansible installed on your local machine.  ----> Note: `LOCAL MACHINE MEANS YOUR OWN COMPUTER NOT THE 3 SERVERS `


## Inventory config


We need to create a Inventory File in our local machine 

_We need to test our inventory with a ping, before to run the next steps._

We need to create a directory named ~/kube-cluster in home directory of your local machine and cd into it:

```sh
mkdir ~/kube-cluster/hosts
```

```sh
nano ~/kube-cluster/hosts
```

```sh
[control_plane]
control1 ansible_host=control_plane_ip ansible_user=root 

[workers]
worker1 ansible_host=worker_1_ip ansible_user=root
worker2 ansible_host=worker_2_ip ansible_user=root

[all:vars]
ansible_python_interpreter=/usr/bin/python3
```

```sh
ansible -i hosts -m ping all
```
