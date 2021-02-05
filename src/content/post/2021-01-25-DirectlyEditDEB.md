+++
title= "DirectlyEditDEB"
date = "2021-01-25T17:17:07+08:00"
description = "DirectlyEditDEB"
keywords = ["Technology"]
categories = ["Technology"]
+++
项目中有时需要对DEB包做些许修改，譬如，在离线场景下某些DEB包的`postinst`中有需要用在线方式下载可执行文件的情况，这种情况下可以借助`dpkg-deb`的解包/压缩命令来临时修改出一个含有所有离线文件的包，举`opennebula-node-firecracker`的包为例说明。    

`opennebula-node-firecracker`是用于在opennebula上运行轻量级VM `firecracker`的包，在线安装时它需要从`github`上拉取相应版本的`firecracker`及 `jailer`可执行文件，这让它在离线场景下无法安装成功。同时，因为它默认作为runtime的一种，其配置文件与`opennebula-node-kvm`里的某些配置文件相同，导致安装时无法继续，我们可以通过如下的方式修改此包，让其适配离线化安装。    

### 准备
在线场景下下载该包，并拷贝到工作目录, 解压:    

```
# cp /var/cache/apt/archives/opennebula-node-firecracker_5.12.0.3-1.ce_amd64.deb .
# mkdir tmp
# dpkg-deb -R opennebula-node-firecracker_5.12.0.3-1.ce_amd64.deb tmp/
```
主要目录结构如下:    

```
├── DEBIAN
│   ├── conffiles
│   ├── control
│   ├── md5sums
│   ├── postinst
│   └── postrm
├── etc
│   ├── cron.d
│   │   └── opennebula-node
│   ├── sudoers.d
│   │   └── opennebula-node-firecracker
│   └── sysctl.d
│       └── bridge-nf-call.conf
├── srv
│   └── jailer
│       └── firecracker
└── usr
    ├── bin
    │   └── svncterm_server
    ├── sbin
    │   ├── install-firecracker
    │   ├── one-clean-firecracker-domain
    │   └── one-prepare-firecracker-domain
    └── share
        └── doc
            └── opennebula-node-firecracker
                ├── changelog.Debian.gz
                ├── copyright
                └── NEWS.Debian.gz
```
我们需要修改的要点如下:    

```
1. DEBIAN/conffiles, 含有此包需写入的配置文件。
2. DEBIAN/md5sums, 含有可执行文件的md5校验码。
3. DEBIAN/postinst, 包安装完成后需执行的脚本。
4. etc/ 下是包安装后在主机上需添加的配置文件。
5. usr/bin, usr/sbin，主机上需拷入的可执行文件。
```

### 修改
观察`DEBIAN/postinst`中含有以下条目:     

```
# cat DEBIAN/postinst 
#!/bin/sh

set -e

ONE_USER=oneadmin

if [ "$1" = "configure" ]; then
    # Install Firecracker + jailer
    /usr/sbin/install-firecracker

```
打开`install-firecracker`文件后观察其下载脚本为:    

```
# cat usr/sbin/install-firecracker 
#!/bin/sh
....

# Download version version of Firecracker
curl -LOJ https://github.com/firecracker-microvm/firecracker/releases/download/${version}/firecracker-${version}-$(uname -m)
mv firecracker-${version}-$(uname -m) /usr/bin/firecracker
chmod +x /usr/bin/firecracker


# Download version version of jailer
curl -LOJ https://github.com/firecracker-microvm/firecracker/releases/download/${version}/jailer-${version}-$(uname -m)
mv jailer-${version}-$(uname -m) /usr/bin/jailer
chmod +x /usr/bin/jailer
```
这里我们直接干掉所有的curl及mv脚本，把预先下载好的`firecracker`/`jailer`文件拷贝到安装目录即可。    

```
# tree ../tmp/usr
../tmp/usr
├── bin
+++ │   ├── firecracker
+++ │   ├── jailer
│   └── svncterm_server
├── sbin
│   ├── install-firecracker
.....
```
此外需要修改md5sums及干掉etc下重复的配置文件。而后压缩包。    

### 压缩包
一条命令:    

```
# dpkg-deb -b tmp opennebula-node-firecracker_5.12.0.3-1.ce_fixed_amd64.deb
```
检查新生成包的大小:      

```
# ls -l -h *.deb
-rw-r--r-- 1 root root  24K Jan 25 09:06 opennebula-node-firecracker_5.12.0.3-1.ce_amd64.deb
-rw-r--r-- 1 root root 1.2M Jan 25 09:11 opennebula-node-firecracker_5.12.0.3-1.ce_fixed_amd64.deb
```
使用`fixed`后的包安装，此时可忽略internet下载过程，且解决了包安装时的冲突问题。   
