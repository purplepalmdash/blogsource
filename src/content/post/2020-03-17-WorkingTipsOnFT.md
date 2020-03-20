+++
title= "WorkingTipsOnFT"
date = "2020-03-17T16:23:03+08:00"
description = "WorkingTipsOnFT"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Tips
官方默认的源是无法安装的

![/images/2020_03_17_16_23_16_910x466.jpg](/images/2020_03_17_16_23_16_910x466.jpg)
备份后换成Ubuntu官方源:    

```
# vim /etc/apt/sources.list
deb http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial main multiverse restricted universe
deb http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-backports main multiverse restricted universe
deb http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-proposed main multiverse restricted universe
deb http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-security main multiverse restricted universe
deb http://mirrors.ustc.edu.cn/ubuntu-ports/ xenial-updates main multiverse restricted universe
# sudo apt-get update -y
```
成功。

### System
Create a vm based on Ubuntu16.04.6, thus you get the qcow2 of the basic images.  Copy another:     

```
# cp ubuntu.qcow2 crack_FT.qcow2
```
Create the disk image via dd:     

```
#  sudo dd if=/dev/sdc | gzip -c > /media/sda/kylin_FT.img
```
Then gunzip it in a very large disk system

### GREEN KFZ
Working. 

```
1. all of the binarys are in portable mode.     
2. ansible in portable.     
3. docker/docker-compose in binary mode.   
4. install-socat in docker images, arm64 distribution. 
5. ignore installing any packages. 

```
