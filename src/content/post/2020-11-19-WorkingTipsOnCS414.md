+++
title= "WorkingTipsOnCS414"
date = "2020-11-19T10:53:10+08:00"
description = "WorkingTipsOnCS414"
keywords = ["Technology"]
categories = ["Technology"]
+++
### System Preparation
Install CentOS7.7, minimal mode, then enable root login, login with root.   

Disable `UseDNS` in `sshd` configuration.    

```
[root@cs ~]# cat /etc/yum.conf 
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=1
# yum upate -y
# yum install vim bridge-utils net-tools -y
```
