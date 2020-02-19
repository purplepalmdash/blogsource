+++
title = "WorkingTipsOnArm64centos7"
date = "2019-07-05T14:57:57+08:00"
description = "WorkingTipsOnArm64centos7"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 仓库配置
使用epel-7及centos7的源为默认源:    

```
[root@arm64 ~]# cat /etc/yum.repos.d/epel7.repo 
[epel]
name=Extra Packages for Enterprise Linux 7 - $basearch
baseurl=http://mirrors.aliyun.com/epel/7/$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
 
[epel-debuginfo]
name=Extra Packages for Enterprise Linux 7 - $basearch - Debug
baseurl=http://mirrors.aliyun.com/epel/7/$basearch/debug
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0
 
[epel-source]
name=Extra Packages for Enterprise Linux 7 - $basearch - Source
baseurl=http://mirrors.aliyun.com/epel/7/SRPMS
failovermethod=priority
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
gpgcheck=0

[root@arm64 ~]# cat /etc/yum.repos.d/centos7.repo 


[base]
name=CentOS-7 - Base
baseurl=https://mirrors.aliyun.com/centos-altarch/7/os/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#released updates 
[updates]
name=CentOS-7 - Updates
baseurl=https://mirrors.aliyun.com/centos-altarch/7/updates/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

#additional packages that may be useful
[extras]
name=CentOS-7 - Extras
baseurl=https://mirrors.aliyun.com/centos-altarch/7/extras/$basearch/
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
enabled=1

#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-7 - Plus
baseurl=https://mirrors.aliyun.com/centos-altarch/7/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7

```

基本情况:    

```
[root@arm64 ~]# uname -a
Linux arm64 4.14.0-115.el7a.0.1.aarch64 #1 SMP Sun Nov 25 20:54:21 UTC 2018 aarch64 aarch64 aarch64 GNU/Linux
[root@arm64 ~]# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (AltArch) 
```

