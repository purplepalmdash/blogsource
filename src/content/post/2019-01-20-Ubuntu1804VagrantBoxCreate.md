+++
title = "Ubuntu1804VagrantBoxCreate"
date = "2019-01-20T11:34:03+08:00"
description = "Ubuntu1804VagrantBoxCreate"
keywords = ["Linux"]
categories = ["Linux"]
+++
For creating Ubuntu 18.04 vagrant box, follow the following steps:    

Change grub configuration for changing to `ethx` naming rules:    

```
# vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quite"
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0 
# grub-mkconfig -o /boot/grub/grub.cfg
```
Change the netplan rules:    

```
# vim /etc/netplan/01-netcfg.yaml 
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: yes
```
Now reboot your machine, continue for later commands.   

Create vagrant user and set the password, etc.   

```
# useradd -m vagrant
# passwd vagrant
# visudo
vagrant ALL=(ALL)	NOPASSWD:ALL
Defaults:vagrant	!requiretty
# mkdir -p /home/vagrant/.ssh
# chmod 0700 /home/vagrant/.ssh/
# wget --no-check-certificate     https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub     -O /home/vagrant/.ssh/authorized_keys
# cat /home/vagrant/.ssh/authorized_keys 
# chmod 0600 /home/vagrant/.ssh/authorized_keys
# chown -R vagrant /home/vagrant/.ssh
# cp /home/test/.bashrc /home/vagrant/.bashrc 
# cp /home/test/.bash_logout /home/vagrant/.bash_logout
# cp /home/test/.profile /home/vagrant/.profile
# vim /home/vagrant/.profile 
add
[ -z "$BASH_VERSION" ] && exec /bin/bash -l
# chsh -s /bin/bash vagrant
```

Finally change the sshd configuration:    

```
# vim /etc/ssh/sshd_config 
AuthorizedKeysFile .ssh/authorized_keys
```

Now you could halt you machine.   

Package to a box via:     

```
$ vagrant package --base xxxx
```
Using the package.box , then you could mutate to libvirt or do some other
things.     

### For kubespray offline
Install ansible via:    

```
# apt-get update && apt-get install -y python-pip && pip install ansibel
``` 
Download kubespray source files:     

```
# wget  https://github.com/kubernetes-sigs/kubespray/archive/v2.8.1.tar.gz
# tar xzvf v2.8.1.tar.gz
# vim inventory/sample/host.ini
[all]
node ansible_host=192.168.33.109 ip=192.168.33.109 etcd_member_name=etcd1

[kube-master]
node

[etcd]
node

[kube-node]
node

[k8s-cluster:children]
kube-master
kube-node
# vim inventory/sample/group_vars/k8s-cluster/k8s-cluster.yml
kube_version: v1.12.4
```
Deploy online for:    

```
# ansible-playbook -i inventory/sample/hosts.ini cluster.yml
```
Now you could get all of the deb packages and docker images. 
