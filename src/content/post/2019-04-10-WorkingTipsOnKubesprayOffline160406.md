+++
title = "WorkingtipsOnUbuntu160406Kubesprayoffline"
date = "2019-04-10T12:10:22+08:00"
description = "WorkingtipsOnUbuntu160406Kubesprayoffline"
keywords = ["Linux"]
categories = ["Technology"]
+++
### VagrantBox
ToBeAdded
### Offline
Steps:    

```
# wget ....../kubespray-2.8.4.tar.gz .
# tar xzvf kubespray-2.8.4.tar.gz 
# cd kubespray-2.8.4
# vim inventory/sample/group_vars/k8s-cluster/addons.yml 
helm_enabled: true
metrics_server_enabled: true
# vim inventory/sample/hosts.ini
[all]
node ansible_host=10.0.2.15  # ip=10.0.2.15 etcd_member_name=etcd1

[kube-master]
node

[etcd]
node

[kube-node]
node

[k8s-cluster:children]
kube-master
kube-node
# cat /etc/apt/sources.list
deb http://mirrors.163.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-backports main restricted universe multiverse
# apt-add-repository ppa:ansible/ansible
# apt-get update
# apt-get install -y ansible python-pip python ntp dbus python-apt
# ansible --version
ansible 2.7.10
  config file = /root/kubespray-2.8.4/ansible.cfg
  configured module search path = [u'/root/kubespray-2.8.4/library']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Nov 12 2018, 14:36:49) [GCC 5.4.0 20160609
# vim /etc/ssh/sshd_config
PermitRootLogin yes
# systemctl restart sshd
# ssh-keygen
# ssh-copy-id root@10.0.2.15
# apt-get install -y bind9 bind9utils ntp nfs-common nfs-kernel-server python-netaddr nethogs iotop
#### find all of the debs and upload it to deployment directory for replacing 1604debs.tar.xz
# ansible-playbook  -i inventory/sample/hosts.ini cluster.yml
```
