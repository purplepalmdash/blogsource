+++
title = "Workingtipsonkubesprayonrhel74"
date = "2019-04-28T16:13:54+08:00"
description = "Workingtipsonkubesprayonrhel74"
keywords = ["Linux"]
categories = ["Linux"]
+++
### System Installation
Install the minimum installation, then set the hostname via:    

```
# hostnamectl set-hostname node1
```
Keep cache via:    

```
# vim /etc/yum.conf
keepcache = 1
```
Disable the subscription plugin:     

```
# vi /etc/yum/pluginconf.d/subscription-manager.conf
enabled = 0
```
### Install ansible
Install epel:    

```
# curl https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm>epel.rpm
# yum install epel.rpm
# yum repolist
```
Mount installation dvd iso:    

```
# mount /dev/sr0 /mnt
# vim /etc/yum.repos.d/local.repo
[local]
name=local
baseurl=file:///mnt
enabled=1
gpgcheck=0
# yum update -y && yum install -y vim python-netaddr
```

Now install ansible via:    

```
# yum install -y ansible
# ansible --version
ansible 2.7.10
  config file = /etc/ansible/ansible.cfg
```
Disable the selinux:    

```
# vim /etc/selinux/config
# setenforce 0
```
Disable the firewalld:    

```
# systemctl disable firewalld
```

Enable the ssh passwordless login:    

```
# ssh-keygen
# ssh-copy-id root@192.168.122.32
```

### Kubespray
Get the kubespray source code, write the inventory file like following:    

```
[all]
node1 ansible_host=192.168.122.32 ansible_ssh_user=root  ip=192.168.122.32

[kube-deploy]
node1

[kube-master]
node1

[bastion]


[calico-rr]


[etcd]
node1

[kube-node]
node1

[k8s-cluster:children]
kube-master
kube-node
```
Deploy via:    

```
# ansible-playbook -i inventory/sample/hosts.ini cluster.yml
```

Failed, should change the 

```
# vim ./roles/bootstrap-os/defaults/main.yml
# vim ./roles/container-engine/docker/defaults/main.yml

changes the releasever to 7, also change the mirror from centos.org to 163.com
or aliyun.com
# rm -f /etc/yum.repos.d/extras.repo
# wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
also change the  releasever to 7
```

Failed , update jinja2:    

```
# yum install python-pip
# pip show jinja2
2.7.x
# pip install jinja2 --upgrade
# pip show jinja2
2.10.1
```

Install more packages:    

```
# yum install -y createrepo iotop parted ntp  nfs-utils bind bind-utils
```


### Install newer docker 
Following are the steps for a brand-new rhel7 vm:    

```
# cd /etc/yum.repos.d
# rm -f redhat.repo
# curl http:/mirrors.163.com/.help/CentOS7-Base-163.repo>base.repo
# vi base.repo
%s#$releasever#7#g
# yum install -y yum-utils device-mapper-persistent-data lvm2
# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# yum search docker-ce
```
Now you could install the specified version of docker-ce and update your
offline pkgs. 
