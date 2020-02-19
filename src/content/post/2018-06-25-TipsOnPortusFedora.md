+++
title = "FedoraPortus"
date = "2018-06-25T10:45:49+08:00"
description = "FedoraPortus"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Steps
Install fedora 28, minimum server.    

```
# dnf install NetworkManager-tui
# vi /etc/selinux/config
SELINUX=disabled
# systemctl disable firewalld
# vi /etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0 rhgb quiet"
# grub2-mkconfig -o /boot/grub2/grub.cfg 
# reboot
# nmtui
```
In `nmtui`, we set its address `192.192.189.188/24`, gateway `192.192.189.1`.    

Install docker-ce for fedora:   

```
# dnf -y install dnf-plugins-core
# dnf config-manager     --add-repo     https://download.docker.com/linux/fedora/docker-ce.repo
# dnf install docker-ce python-pip
# pip install docker-compose
```
Transfer to fedora 28. 
