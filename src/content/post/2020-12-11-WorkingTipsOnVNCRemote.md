+++
title= "WorkingTipsOnVNCRemote"
date = "2020-12-11T08:44:50+08:00"
description = "WorkingTipsOnVNCRemote"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Hardware & OS
Hardware configuration:  

```
# lscpu
Intel(R) Xeon(R) CPU E5-2650 v2 @ 2.60GHz
32Core
# free -g
              total        used        free      shared  buff/cache   available
Mem:             62          19          10           0          33          42
Swap:             0           0           0
# df -h
/dev/mapper/vg-root  1.7T  1.1T  538G  66% /
```
OS Configuration:     

```
# cat /etc/issue
Ubuntu 16.04.4 LTS \n \l
```
### AIM
To use this server as the vagrant environment.    

### vagrant-libvirt
use docker for running vagrant:    

```
# docker pull vagrantlibvirt/vagrant-libvirt:latest
```
Install libvirtd related:   

```
# apt-get install -y virt-manager
# systemctl status libvirt-bin qemu
```
### Desktop
Use awesome as the default desktop:    

```
# apt-get install -y i3
# /usr/lib/apt/apt-helper download-file https://debian.sur5r.net/i3/pool/main/s/sur5r-keyring/sur5r-keyring_2020.02.03_all.deb keyring.deb SHA256:c5dd35231930e3c8d6a9d9539c846023fe1a08e4b073ef0d2833acd815d80d48
# dpkg -i ./keyring.deb
# echo "deb http://debian.sur5r.net/i3/ $(grep '^DISTRIB_CODENAME=' /etc/lsb-release | cut -f2 -d=) universe" >> /etc/apt/sources.list.d/sur5r-i3.list
# apt-get update -y
# apt install i3
# apt-get install -y tigervncserver
# vncpasswd
# vncserver -localhost -nolisten tcp
# vim ~/.vnc/xstartup
#!/bin/bash
i3 &
```
Change to lxde4:   

```
cat ~/.vnc/xstartup
	#/etc/X11/Xsession
	exec startlxde

```
### client
Enable the ssh transfering:    

```
$ ssh -p 62022 -L 127.0.0.1:5901:localhost:5901 root@xxx.xxx.xxx.xxx
```
then viewer `localhost:5901` you could see the desktop
