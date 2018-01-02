+++
categories = ["Virtualization"]
date = "2016-11-30T15:00:57+08:00"
description = "BuildRPMsOnXenServer"
keywords = ["Virtualization"]
title = "BuildCollectdForXenServer"

+++
### 先决条件
启用aliyun CentOS源，安装工具:    

```
$ yum install rpm-build --skip-broken
```
因为编译可能会占用大量硬盘空间，预先加载某个NFS卷:    

```
# mount -t nfs 192.168.0.221:/xxxx /mnt
```
下载源码包:    

错误！！！ rpm-build安装失败。    

不建议在xenserver上手动编译，上网搜索，找到collectd正确的源:     

```
# vim /etc/yum.repos.d/collectd-ci.repo
[collectd-ci]
name=collectd CI
baseurl=http://pkg.ci.collectd.org/rpm/collectd-5.5/epel-5-$basearch
enabled=1
gpgkey=http://pkg.ci.collectd.org/pubkey.asc
gpgcheck=0
repo_gpgcheck=0
# yum remove collectd && yum install -y collectd
```

### Trouble-Shooting
如果激活有其他源，则可能会因为优先级顺序，优先安装例如epel里的collectd-i386之类的包，
解决方案是将这些源全盘屏蔽。    

```
[root@xenserver-WolfHunter yum.repos.d]# ls
back  Citrix.repo  collectd-ci.repo
```
