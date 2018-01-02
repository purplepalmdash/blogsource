+++
date = "2017-03-28T15:50:51+08:00"
title = "Working tips on CloudStack49 On CentOS7"
categories = ["Virtualization"]
keywords = ["Cloudstack"]
description = "Install cloudstack4.9 on CentOS7"

+++
### Preparation
Upgrading to newest version of centos7:    

```
# yum update -y
```
Added epel library and cloudstack 4.9 repository:    

```
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# vim /etc/yum.repos.d/cloudstack.repo
[cloudstack]
name=cloudstack
baseurl=http://cloudstack.apt-get.eu/centos/7/4.9/
gpgcheck=0
enabled=1
# yum makecache
```
Enable the yum's cache so you could save the cached rpms into local directory
and then install following packages:    

```
# yum install -y vim bridge-utils wget net-tools ntp cloudstack-management
mariadb-server mariadb cloudstack-agent cloudstack-cli cloudstack-usage
createrepo python-pip nethogs iotop
```
Get the pip cache for cloudmonkey, for later we will use cloudmonkey for
configurating the cloudstack:    

```
# pip install cloudmonkey --download=/root/pipcache/
# pip install --no-index --find-links ~/pipcache cloudmonkey
```
Now copy all of the rpms and the pip caches to your own webserver.    

Generate the rpm repository via following command:   

```
# cd /root/cache
# mkdir -p /root/rpms
# find . | \grep rpm$ | xargs -I % cp % /root/rpms
# cd /root/rpms/
# createrepo .
```
