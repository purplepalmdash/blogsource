+++
title = "RaspberryPIArm64"
date = "2019-06-26T11:03:46+08:00"
description = "RaspberryPIArm64"
keywords = ["Linux"]
categories = ["Technology"]
+++
### AIM
Replace the armhf(raspbain) with arm64 system.   
Refers to:     

[/home/dash/Code/blogsource-master/src/content/post/2019-06-26-RaspberryPIArm64.md](/home/dash/Code/blogsource-master/src/content/post/2019-06-26-RaspberryPIArm64.md)    
### Installation
Unxz the images and write to the tf card:     

```
# unxz ubuntu-18.04.2-preinstalled-server-arm64+raspi3.img.xz
# sudo dd if=./ubuntu-18.04.2-preinstalled-server-arm64+raspi3.img of=/dev/sdd bs=1M && sudo sync
```
### Configuration
Configure the repository :     

```
# vim /etc/apt/sources.list
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic main main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-updates main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-updates main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-backports main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-backports main restricted universe multiverse
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-security main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-security main restricted universe multiverse
# sudo apt-get update -y
# sudo apt-get upgrade -y
# sudo hostnamectl set-hostname arm64
# sudo apt-get install -y build-essential
```

Install docker-ce for arm64:     

```
#  offline packages. 
# apt-get install -y docker-ce
# docker version
# cd /var/lib
# cp -r docker /media/sda/
# mv docker docker.back
# ln -s /media/sda/docker .
# ls -l -h | grep docker
lrwxrwxrwx  1 root      root        17 Jun 26 04:16 docker -> /media/sda/docker
drwxr-xr-x  2 root      root      4.0K Jun 26 04:13 docker-engine
drwx--x--x 14 root      root      4.0K Jun 26 04:13 docker.back
# systemctl start docker
```

Building harbor:     

```
# apt-get install -y docker-compose

```
dns issue, install stubby:    

```
# apt-get install -y stubby
# vim /etc/resolv.conf
nameserver 127.0.0.1
```

