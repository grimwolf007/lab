# Setup Nodes
 - [ ] Centos 8 Stream
   - [ ] master
   - [ ] node1
   - [ ] node2
 - [ ] Ubuntu Server
   - [ ] minikube
 
- [ ] Setup hostname (centos)
  - `sudo hostnamectl set-hostname host.domain` 
- [ ] Setup Networking
 - `nmtui` - on centos 8 stream
 - `netplan` - on ubuntu 20.0.4

# [Minikube](https://minikube.sigs.k8s.io/docs/start/)
 - [ ] setup docker repo
  ```
   sudo apt-get update
   sudo apt-get install \
      ca-certificates \
      curl \
      gnupg \
      lsb-release
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
  echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
- [ ] install docker
  ```
   sudo apt-get update
   sudo apt-get install docker-ce docker-ce-cli containerd.io
  ```
 - [ ] install deb package 
  ```
  curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb
  sudo dpkg -i minikube_latest_amd64.deb
  ```
 - [ ] start docker and run on start-up ` sudo systemctl start docker; sudo systemctl enable docker`
 - [ ] Add user to docker group *UNSAFE* `sudo usermod -aG docker $USER && newgrp docker`
# k8s

- [ ] Remove gui on centos
  ```
  dnf groupinstall -y "Minimal Install"
  dnf groupremove -y "Server with GUI"
  dnf groupinstall -y "Server"
  ```

# [Install docker](https://docs.docker.com/engine/install/centos/)
 - [ ] setup repo
    ```
    sudo yum install -y yum-utils
    sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    ```
 - [ ] install docker `sudo yum install docker-ce docker-ce-cli containerd.io --allowerasing`
 - [ ] start docker and make it run on reboot `sudo systemctl start docker; sudo systemctl enable docker`
 - [ ] use systemd as cgroup
 ```
  cat << EOF >> /etc/docker/daemon.json
  
  {
    "exec-opts": ["native.cgroupdriver=systemd"]
  }
  EOF
  ``` 
# Install kubeadm
 - [ ] Verify Mac Addresses are different `ip link`
 - [ ] Ensure UUIDs are different `sudo cat /sys/class/dmi/id/product_uuid`
 - [ ] Ensure bridged filtering is enabled `lsmod | grep br_netfilter` If not available do below
    ```
   cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
   br_netfilter
   EOF

   cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   sudo sysctl --system 
   ```
 - [ ] Ensure swap memory is disabled `sudo swapoff -a`
 - [ ] [Comment out swap in /etc/fstab](https://www.tecmint.com/disable-swap-partition-in-centos-ubuntu/)

# Open firewall
 - [ ] Enable masquerading (moving traffic between network interfaces) 
  ```
  firewall-cmd --add-masquerade --permanent
  firewall-cmd --reload
  ```
 - [ ] On master
  ```
  firewall-cmd --zone=public --permanent --add-port={6443,2379,2380,10250,10251,10252}/tcp
  firewall-cmd --reload
  firewall-cmd --zone=public --permanent --add-rich-rule 'rule family=ipv4 source address=10.0.1.22/32 accept'
  firewall-cmd --zone=public --permanent --add-rich-rule 'rule family=ipv4 source address=10.0.1.23/32 accept'
  firewall-cmd --reload
  ```
 - [ ] On worker
  ```
  firewall-cmd --zone=public --permanent --add-port={10250,30000-32767}/tcp
  firewall-cmd --reload
  ```


## As Master

- [ ] [Install kubeadm](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
- [ ] Ensure required ports are open `telnet 127.0.0.1 6443`
- [ ] Kill SELinux and install kubeadm (creates the cluster), kubelet (the cluster server), kubectl (the cluster client)
  ```
  cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
  [kubernetes]
  name=Kubernetes
  baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
  enabled=1
  gpgcheck=1
  repo_gpgcheck=1
  gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
  exclude=kubelet kubeadm kubectl
  EOF
  
  # Set SELinux in permissive mode (effectively disabling it)
  sudo setenforce 0
  sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
  
  sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
  
  sudo systemctl enable --now kubelet
  ```


# [Setup Cluster](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
##  FOR k8s 1.21: [Configure cgroup driver](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/) (systemd) 
  ```
  cat << EOF > kubeadm-config.yaml
  kind: ClusterConfiguration
  apiVersion: kubeadm.k8s.io/v1beta3
  kubernetesVersion: v1.21.0
  ---
  kind: KubeletConfiguration
  apiVersion: kubelet.config.k8s.io/v1beta1
  cgroupDriver: systemd
  EOF
  kubeadm init --config kubeadm-config.yaml
  ```
## FOR k8s >= 1.22
`kubeadm init`
 - [ ] Save the command given by kubeadm init.

## [Install a CNI](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) (Cluster Network Interface)  (I will be using WEAVE)
`kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"`

## Join the nodes
 - [ ] enable kubelet `systemctl enable kubelet`
 - [ ] Run the command given by kubeadm init
 - OR
 - on the master:
 - [ ] `kubeadm token list` = token
   - [ ] If you want to join a new node after 24h, you need to recreate a token `kubeadm token create`
 - [ ] `openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //' ` = hash

 - on the nodes:
 - [ ] (command is given by the init command) `kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>`


