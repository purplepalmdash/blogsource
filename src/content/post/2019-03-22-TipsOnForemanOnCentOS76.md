+++
title = "TipsOnForemanOnCentOS76"
date = "2019-03-22T17:32:15+08:00"
description = "TipsOnForemanOnCentOS76"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Prerequites
System infos:    

```
# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core)
# vi /etc/yum.conf
keepcache=1
#### Configure yum.conf and install some packages
# yum install -y wget vim net-tools gcc
#  vim /etc/seliconfig/config
SELINUX=disabled
# systemctl diable firewalld
```
### Install foreman
Configure host first:    

```
# hostnamectl set-hostname foreman.fuck.com
# vim /etc/hosts
10.192.192.2	foreman.fuck.com
```
Networking configuration like following(using nmtui):     

![/images/2019_03_22_17_42_55_626x250.jpg](/images/2019_03_22_17_42_55_626x250.jpg)

Install via:    

```
# sudo yum -y install https://yum.puppetlabs.com/puppet5/puppet5-release-el-7.noarch.rpm
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# sudo yum -y install https://yum.theforeman.org/releases/1.21/el7/x86_64/foreman-release.rpm
# sudo yum -y install foreman-installer

```
