+++
title= "RPICluster"
date = "2021-02-05T16:08:11+08:00"
description = "RPICluster"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Flashing
Flash with following os:    

```
root@ubuntu:/home/ubuntu# uname -a
Linux ubuntu 5.4.0-1028-raspi #31-Ubuntu SMP PREEMPT Wed Jan 20 11:30:45 UTC 2021 aarch64 aarch64 aarch64 GNU/Linux
root@ubuntu:/home/ubuntu# cat /etc/issue
Ubuntu 20.04.2 LTS \n \l
```

Disable auto update:    

```
# systemctl stop apt-daily.timer;systemctl disable apt-daily.timer ; systemctl stop apt-daily-upgrade.timer ; systemctl disable apt-daily-upgrade.timer; systemctl stop apt-daily.service;  systemctl mask apt-daily.service; systemctl daemon-reload
```
Configure hostname:    

```
# hostnamectl set-hostname rpi1
# hostnamectl set-hostname rpi2
```
Configure cn repository:    

```
# vim /etc/apt/sources.list
deb http://mirrors.ustc.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ focal main main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
deb http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
# deb-src http://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
# apt-get update -y
```
Install some packages:    

```
# apt-get install -y net-tools virt-manager lxde tightvncserver sshpass ssh-askpass
# usermode -a -G kvm,libvirt  ubuntu
```
Configure vnc

```
# vncserver
# vim ~/.vnc/xstartup
	#!/bin/sh
	exec startlxde
# vncserver -kill :1
# vncserver
```
In vnc we could use virt-manager.    

Install new os:    

![/images/2021_02_05_16_53_45_633x380.jpg](/images/2021_02_05_16_53_45_633x380.jpg)


