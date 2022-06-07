# HomeInfra

## Kubernetes setup

### Master Node
* Firewall config
  ```bash
  firewall-cmd --permanent --add-port=80/tcp
  firewall-cmd --permanent --add-port=443/tcp
  firewall-cmd --permanent --add-port=2376/tcp
  firewall-cmd --permanent --add-port=2379/tcp
  firewall-cmd --permanent --add-port=2380/tcp
  firewall-cmd --permanent --add-port=6443/tcp
  firewall-cmd --permanent --add-port=8472/udp
  firewall-cmd --permanent --add-port=9099/tcp
  firewall-cmd --permanent --add-port=10250/tcp
  firewall-cmd --permanent --add-port=10254/tcp
  firewall-cmd --permanent --add-port=30000-32767/tcp
  firewall-cmd --permanent --add-port=30000-32767/udp
  firewall-cmd --permanent --add-interface=cni0 --zone=trusted
  firewall-cmd --reload
  ```
* SELinux config
  ```bash
  yum install -y container-selinux selinux-policy-base
  ```

* Install K3s
  ```bash
  curl -sfL https://get.k3s.io | sh -
  ```

* Retrieve Token
  This is used later as the value for `SECRET_TOKEN`
  ```bash
  cat /var/lib/rancher/k3s/server/node-token
  ```

* Retrieve Certificate
  This is placed in `~/.kube/k3s.yaml`
  ```bash
  cat /etc/rancher/k3s/k3s.yml
  ```

### Worker Nodes
* Firewall config
  ```bash
  firewall-cmd --permanent --add-port=22/tcp
  firewall-cmd --permanent --add-port=80/tcp
  firewall-cmd --permanent --add-port=443/tcp
  firewall-cmd --permanent --add-port=2376/tcp
  firewall-cmd --permanent --add-port=8472/udp
  firewall-cmd --permanent --add-port=9099/tcp
  firewall-cmd --permanent --add-port=10250/tcp
  firewall-cmd --permanent --add-port=10254/tcp
  firewall-cmd --permanent --add-port=30000-32767/tcp
  firewall-cmd --permanent --add-port=30000-32767/udp
  firewall-cmd --permanent --add-interface=cni0 --zone=trusted
  firewall-cmd --reload
  ```
  
  * SELinux config
  ```bash
  yum install -y container-selinux selinux-policy-base
  ```

* Install K3s
  ```bash
  SECRET_TOKEN='<value from the retrieve token step during master config>'
  curl -sfL https://get.k3s.io | K3S_URL=https://192.168.0.150:6443 K3S_TOKEN=$SECRET_TOKEN sh -
  ```
