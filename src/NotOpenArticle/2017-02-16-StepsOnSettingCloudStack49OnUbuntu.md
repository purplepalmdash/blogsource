+++
date = "2017-02-16T15:35:30+08:00"
title = "CloudStack4.9 On Ubuntu16.04"
categories = ["Virtualization"]
keywords = ["Linux"]
description = "Setup CloudStack 4.9 On Ubuntu16.04"

+++
### 先决条件
Ubuntu16.04安装
root密码设置好，允许root通过ssh登录.

```
# sed -i '/PermitRootLogin without-password/PermitRootLogin yes' /etc/ssh/sshd_config
# service ssh restart
```

准备安装文件,
首先从`http://cloudstack.apt-get.eu/ubuntu/`下载相应的deb包，然后通过`dpkg-scanpackages
. /dev/null | gzip -9c > Packages.gz`命令设置成本地安装源以加速安装。

#### Disable apparmor
步骤:    

```
 apt-get --purge remove apparmor apparmor-utils
```
#### 关闭/删除防火墙
关闭并检查状态:    

```
# ufw disable
# ufw status verbose
```

### 安装cloudstack-management
#### 主机名
需要配置好主机名:    

```
# hostname --fqdn
allinone
```
#### ntp配置
安装openntpd:   

```
# apt-get install -y openntpd
```
#### 网络配置
配置桥接网络,
我的机器上用eth1桥接到cloudbr0，一般的机器上可能是eth0之类，根据自己的需要灵活改动:    

```
# apt-get install -y bridge-utils
# vim /etc/network/interfaces
```
其文件内容如下:    

```
auto eth1
iface eth1 inet manual

# Public network
auto cloudbr0
iface cloudbr0 inet static
    address 10.168.88.100
    netmask 255.255.255.0
    gateway 10.168.88.1
    bridge_ports eth1
    bridge_fd 5
    bridge_stp off
    bridge_maxwait 1

# Private network
auto cloudbr1
iface cloudbr1 inet manual
    bridge_ports none
    bridge_fd 5
    bridge_stp off
    bridge_maxwait 1
```
#### 安装cloudstack-management包
通过添加本地的安装源后，用如下命令安装:    

```
root@allinone:/etc/apt/sources.list.d# cat cloudstack.list 
deb http://192.168.1.69/cloudstack49deb/ 	/
```
安装:    

```
# apt-get update
# apt-get install cloudstack-management
```
#### 修改jre加密
openjdk8的加密使用`/dev/random`，会造成创建数据库时卡在`Process
encryption`这一步，解决方案如下

```
# vim /etc/java-8-openjdk/security/java.security
# vim /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/security/java.security
```
在其中替换:    

```
- securerandom.source=file:/dev/random
+ securerandom.source=file:/dev/urandom
```

#### 安装mysql
安装mysql的命令为`apt-get install mysql-server`，记得输入两次密码.    

更改配置文件:    

```
# vim /etc/mysql/mysql.conf.d/mysqld.cnf
[mysqld]
//.....
datadir = /var/lib/mysql
server-id=master-01
innodb_rollback_on_timeout=1
innodb_lock_wait_timeout=600
max_connections=350
log-bin=mysql-bin
binlog-format = 'ROW'
```
重新启动mysql服务器:    

```
# service mysql restart
```
#### 创建数据库
通过下列命令创建数据库, 其中两个`engine123`分别为数据库密码:

```
# cloudstack-setup-databases cloud:engine123@localhost --deploy-as=root:engine123 -e file -m mymskey44 -k mydbkey00
```

#### 准备存储
准备共享目录及安装nfs服务器:    

```
# mkdir -p /export/primary /export/secondary
# apt-get install nfs-kernel-server
```
引出`/export`目录:    

```
# vim /ec/exports
/export  *(rw,async,no_root_squash,no_subtree_check)
# exportfs -a
# showmount -e 127.0.0.1
```
### 安装cloudstack-agent
安装:    

```
# apt-get install cloudstack-agent
```
配置libvirt:    

```
# vim /etc/libvirt/libvirtd.conf
listen_tls = 0
listen_tcp=1
tcp_port="16509"
auth_tcp="none"
```
配置`libvirt-bin.conf`文件:    

```
# vim /etc/default/libvirt-bin
libvirtd_opts="-d -l"
```
配置`qemu.conf`文件，以侦听所有端口:     

```
# vim /etc/libvirt/qemu.conf
vnc_listen = "0.0.0.0"
```

### ISSUE
Ubuntu16.04上没有tomcat6的包，导致cloudstack-management安装有问题。    

方案：   
A, 在Ubuntu16.04上安装tomcat6.    
B, 采用Ubuntu14.04作为management服务器.    

### cloudmonkey issue
The cloudmonkey could be installed via following commands:    

```
$ sudo apt-get install cloudstack-cli
$ sudo apt-get install python-pip
$ pip install cloudmonkey
$ sudo easy_install --upgrade requests
$ sudo easy_install --upgrade pygments
```
then you could use cloudmonkey.    
