# Build a Kubernetes cluster using k3s via Ansible

Author: <https://github.com/itwars>

## K3s Ansible Playbook

Build a Kubernetes cluster using Ansible with k3s. The goal is easily install a Kubernetes cluster on machines running:

- [X] Debian
- [X] Ubuntu
- [X] CentOS/RHEL/Fedora

on processor architecture:

- [X] x64
- [X] arm64
- [X] armhf

## System requirements

Deployment environment must have Ansible 2.4.0+
Master and nodes must have passwordless SSH access

## Usage

* Edit `inventory/k3s/hosts.ini` to match the system. For example:

  ```bash
  [master]
  192.16.35.12

  [node]
  192.16.35.[10:11]

  [k3s_cluster:children]
  master
  node
  ```

  If needed, you can also edit `inventory/k3s/group_vars/all.yml` to match your environment.

* Setup authentication to each node. 
  * Note this assumes you already have an [ssh key created](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

  Run the following for each replacing the `X` values with the node's IP address.
  ```bash
  ssh-copy-id christ@192.168.0.XXX
  ```

* Start provisioning of the cluster using the following command:

  ```bash
  ansible-playbook site.yml -i inventory/k3s/hosts.ini -u christ --become --ask-become-pass
  ```

## Kubeconfig

* To get access to your **Kubernetes** cluster just

  ```bash
  scp debian@master_ip:~/.kube/config ~/.kube/config
  ```
