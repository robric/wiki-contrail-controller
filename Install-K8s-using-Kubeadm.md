# Installing Kubernetes on Master VM (Centos)

sudo setenforce 0

Install Docker
```
sudo yum install -y docker
systemctl enable docker.service;service docker start

```

sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

sudo echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update;sudo swapoff -a

sudo apt-get install -y kubectl kubelet kubeadm docker-engine

sudo vi /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

Comment out "#Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin"
sudo systemctl daemon-reload;sudo service kubelet restart

kubectl label node node-role.opencontrail.org/controller=true

sudo sysctl -w net.bridge.bridge-nf-call-iptables=1

sudo systemctl stop firewalld; sudo systemctl disable firewalld

kubeadm init

Once "kubeadm init" completes, copy the output which would be like below:

"kubeadm join 192.168.1.3:6443 --token 0smq4g.7pmg2jqc8arl1uz7 --discovery-token-ca-cert-hash sha256:d92ac0785b1435666d726f4bc54fde58693f87cf91371d9fd553da4a40813650"

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config

Put "sudo chown $(id -u):$(id -g) $HOME/.kube/config” in to bash

add "sudo chown $(id -u):$(id -g) $HOME/.kube/config”