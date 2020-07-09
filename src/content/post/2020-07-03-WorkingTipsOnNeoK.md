+++
title= "WorkingTipsOnNeoK"
date = "2020-07-03T09:26:11+08:00"
description = "WorkingTipsOnNeoK"
keywords = ["Technology"]
categories = ["Technology"]
+++
某国产操作系统，安装手记，不要问为啥写这么没技术含量的东西，因为ZHENGCE需要上，某些公司要赚经费罢了，安可是门生意，仅此而已。

### 安装
无他，virt-manager里，ISO安装，最小化，安装完毕重新启动。  

装完一看，果然:    

```
# cat /etc/redhat-release
Red Hat Enterprise Linux Server release 7.6 (Maipo)
```

### 配置
ISO挂载:    

```
# mount /dev/sr0 /mnt
# vim /etc/yum.repos.d/local.repo
[local]
name=local
baseurl=file:///mnt
enabled=1
gpgcheck=0
# mv /etc/yum.repos.d/ns7-adv.repo /root
# yum update
```

安装必要的包(跟rhel 7.6完全一样嘛):    

```
# yum groupinstall "Server with GUI"
# yum install -y tigervnc-sever git gcc gcc-c++ java-11-openjdk java-11-openjdk-devel iotop 
# vncserver
# systemctl stop firewalld && systemctl disable firewalld
```
此时进去以后是vncviewer的桌面，和RHEL一模一样。     

外网搞一个xrdp的包进来:     

```
# apt-get install -y docker.io
# docker pull centos:7
# docker run -it centos:7 /bin/bash
# vim /etc/yum.conf
keepcache=1
# yum install -y epel-releases
# yum update 
# yum install -y xrdp
```
安装/启动xrdp

```
# yum install -y xrdp
# systemctl enable xrdp
```
去掉授权:    

```
# mv /etc/xdg/autostart/licmanager /root
```
Now you could use it.   
Tranform it into lxc and run lxc 
