+++
title = "KubesprayQuickOffline"
date = "2019-05-05T10:11:40+08:00"
description = "KubesprayQuickOffline"
keywords = ["Linux"]
categories = ["Technology"]
+++
### OS prepare
Download the `tar.gz` file from github, untar, then modify the Vagrantfile
with:    

```
# vim cluster1.yml
---
- hosts: all
  gather_facts: false
  become: True
  tasks:
    - name: "Run shell"
      shell: uptime
# vim Vagrantfile
SUPPORTED_OS = {

  "wukong"              => {box: "xxxx160406", user: "vagrant"},
}

$os = "wukong"

$playbook = "cluster1.yml"
# rm -rf .vagrant
# rm -f inventory/sample/vagrant_ansible_inventory
# vagrant up
```
### Get Packages/Images
ssh into the deployed vagrant machine,  set the networking with firewall-less
networking.   

Configure the repository:    

```
# vim /etc/apt/sources.list
deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse
```

Install ansible:    

```
# apt-add-repository ppa:ansible/ansible
# apt-get update -y
# apt-get install -y ansible python-pip python ntp dbus python-apt bind9 bind9utils ntp nfs-common nfs-kernel-server python-netaddr nethogs iotop
``` 

Permit root nopasswrd login:    

```
# vim /etc/ssh/sshd_config
PermitRootLogin yes
# systemctl restart sshd
# ssh-keygen
# ssh-copy-id root@172.17.48.101
```
Copy the Vagrant host's kubespray folder into your vm folder, change the
`inventory/sample/vagrant_ansible_inventory` to :    

```
k8s2110-1 ansible_host=172.17.48.101 ansible_port=22 ansible_user='root' ip=172.17.48.101 flannel_interface=eth1 kube_network_plugin=calico kube_network_plugin_multus=false ansible_ssh_user=root

[etcd]
k8s2110-[1:1]

[kube-master]
k8s2110-[1:1]

[kube-node]
k8s2110-[1:1]

[k8s-cluster:children]
kube-master
kube-node
```

Now deploy it via:    

```
# ansible-playbook -i inventory/sample/vagrant_ansible_inventory cluster.yml
```
kubespray ansible will pull additional packages, also the docker images, for
setting up the kubernetes cluster.    

Generate the packages:    

```
# mkdir -p /root/debs
# cd /var/cache
# find . | grep deb$ | xargs -I % cp % /root/debs/
# cd /root/
# mv debs static
# cd static
# dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
# cd ..
# tar cJvf 1604debs.tar.xz static
```
Transfer the 1604debs.tar.xz for uploading.    

Save images via:    

```
# docker save -o 2110.tar nginx:1.15 gcr.io/google_containers/metrics-server-amd64:v0.3.2 gcr.io/google-containers/kube-proxy:v1.14.1 gcr.io/google-containers/kube-apiserver:v1.14.1 gcr.io/google-containers/kube-controller-manager:v1.14.1 gcr.io/google-containers/kube-scheduler:v1.14.1 coredns/coredns:1.5.0 lachlanevenson/k8s-helm:v2.13.1 gcr.io/kubernetes-helm/tiller:v2.13.1 k8s.gcr.io/cluster-proportional-autoscaler-amd64:1.4.0 k8s.gcr.io/k8s-dns-node-cache:1.15.1 gcr.io/google-containers/coredns:1.3.1 quay.io/coreos/etcd:v3.2.26 gcr.io/google_containers/kubernetes-dashboard-amd64:v1.10.1 calico/node:v3.4.0 calico/cni:v3.4.0 calico/kube-controllers:v3.4.0 rancher/local-path-provisioner:v0.0.2 nfvpe/multus:v3.1.autoconf k8s.gcr.io/addon-resizer:1.8.3 quay.io/external_storage/local-volume-provisioner:v2.1.0 gcr.io/google-containers/pause:3.1 gcr.io/google_containers/pause-amd64:3.1
```
Use 2110.tar for replacing the secureregistryserver's content.   

Upload some files to the `kube-deploy/files`:    

```
hyperkube
kubeadm
calicoctl
cni-plugins-amd64-v0.6.0.tgz
```

### 18.04.02
Use another 18.04.02 vagrant box for deploying, after installation, just zip
to another tar.xz file:    

```
# tar cJvf 1804debs.tar.xz static/
```

### kubadm ssl issue
Change the ssl issue needs do modification to source code and re-generate
kubeadm.  Use the new-generated kubeadm for replacing the origin ones, and
replaces its sha256sum.    
