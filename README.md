#  Kubernetes Cluster Using Kubeadm on Ubuntu 20.04


## Prerequisites

- SSH accces configuration for the machines
- 3 servers running Ubuntu 20.04 with at least 2GB RAM and 2 vCPUs.  
- Ansible installed on your local machine.  ----> Note: `LOCAL MACHINE MEANS YOUR OWN COMPUTER NOT THE 3 SERVERS `




Disable Swap and Enable Kernel modules

```sh
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapoff -a
```

Next we need to enable kernel modules and configure sysctl.

To enable kernel modules
```sh
sudo modprobe overlay
sudo modprobe br_netfilter
```
Add some settings to sysctl

```sh
sudo tee /etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF
```
Reload sysctl

```sh
sudo sysctl --system
```
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

## Dependency Installing

```sh
nano ~/kube-cluster/kube-dependencies.yml
```

```sh
---
- hosts: all
  become: yes
  tasks:
   - name: create Docker config directory
     file: path=/etc/docker state=directory

   - name: changing Docker to systemd driver
     copy:
      dest: "/etc/docker/daemon.json"
      content: |
        {
        "exec-opts": ["native.cgroupdriver=systemd"]
        }

   - name: install Docker
     apt:
       name: docker.io
       state: present
       update_cache: true

   - name: install APT Transport HTTPS
     apt:
       name: apt-transport-https
       state: present

   - name: add Kubernetes apt-key
     apt_key:
       url: https://packages.cloud.google.com/apt/doc/apt-key.gpg
       state: present

   - name: add Kubernetes' APT repository
     apt_repository:
      repo: deb http://apt.kubernetes.io/ kubernetes-xenial main
      state: present
      filename: 'kubernetes'

   - name: install kubelet
     apt:
       name: kubelet=1.22.4-00
       state: present
       update_cache: true

   - name: install kubeadm
     apt:
       name: kubeadm=1.22.4-00
       state: present

- hosts: control_plane
  become: yes
  tasks:
   - name: install kubectl
     apt:
       name: kubectl=1.22.4-00
       state: present
       force: yes
```



```sh
---
- hosts: control_plane
  become: yes
  tasks:
    - name: initialize the cluster
      shell: kubeadm init --pod-network-cidr=10.244.0.0/16 >> cluster_initialized.txt
      args:
        chdir: $HOME
        creates: cluster_initialized.txt

    - name: create .kube directory
      become: yes
      become_user: ubuntu
      file:
        path: $HOME/.kube
        state: directory
        mode: 0755

    - name: copy admin.conf to user's kube config
      copy:
        src: /etc/kubernetes/admin.conf
        dest: /home/ubuntu/.kube/config
        remote_src: yes
        owner: ubuntu

    - name: install Pod network
      become: yes
      become_user: ubuntu
      shell: kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml >> pod_network_setup.txt
      args:
        chdir: $HOME
        creates: pod_network_setup.txt
        
  ```
