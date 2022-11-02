+++
title= "MigrationFromCentOS76ToRocky"
date = "2022-11-02T14:05:53+08:00"
description = "MigrationFromCentOS76ToRocky"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. 初始环境(CentOS76)
使用vagrant创建`CentOS7.6.1810`虚拟机一台:     

```
# vim Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7.6"
    config.vm.provider "virtualbox" do |v|
        v.memory = 4096
        v.cpus = 4
    end
end
# vagrant up 
# vagrant ssh
```
登陆入系统后检查初始环境信息:    

```
[vagrant@localhost ~]$ cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
[vagrant@localhost ~]$ uname -a
Linux localhost.localdomain 3.10.0-957.21.3.el7.x86_64 #1 SMP Tue Jun 18 16:35:19 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```
### 2. CentOS7.6升级到CentOS8.1

#### 2.1 升级前准备
安装EPEL(Extra Package for Enterprise Linux)库，因升级中需用到该库, 并创建该库缓存:     

```
# yum install -y epel-release.noarch
# yum makecache fast
```
安装升级CentOS 7到8过程中需要用到的`rpmconf`包及`yum-utils`包:    

```
# yum install -y yum-utils rpmconf
```
计算重复/无用包及配置:    

```
# rpmconf -a
安装过程中，选择 N:  
*** aliases (Y/I/N/O/D/M/Z/S) [default=N] ? 
Your choice: N
```
升级`yum`为`dnf`，安装完毕后，删除不再需要的yum及相关包：    

```
# yum install -y dnf
# dnf remove -y yum yum-metadata-parser
```
删除现有的repo配置并重新生成仓库缓存, 并使用dnf升级为最新状态:     

```
# rm -Rf /etc/yum
#  dnf makecache
Extra Packages for Enterprise Linux 7 - x86_64   5.9 MB/s |  16 MB     00:02    
CentOS-7 - Base                                  4.1 MB/s |  10 MB     00:02    
CentOS-7 - Updates                               7.6 MB/s |  21 MB     00:02    
CentOS-7 - Extras                                165 kB/s | 332 kB     00:02    
Metadata cache created.
# dnf upgrade -y
```
#### 2.2 升级8仓库准备
跨大版本升级可能会衍生出一些问题，因而我们先使用8.1版本的CentOS8系统包作为中间状态升级:     

```
# dnf upgrade -y  https://mirrors.ustc.edu.cn/centos-vault/8.1.1911//BaseOS/x86_64/os/Packages/{centos-release-8.1-1.1911.0.8.el8.x86_64.rpm,centos-gpg-keys-8.1-1.1911.0.8.el8.noarch.rpm,centos-repos-8.1-1.1911.0.8.el8.x86_64.rpm}
```
升级EPEL仓库包从EL 7到EL 8:      

```
# dnf upgrade -y epel-release
```
因跨大版本升级可能会导致问题，切换`BaseOS`及`AppStream`库为`8.1.1911`:    

```
# cd /etc/yum.repos.d
# mkdir back
# mv CentOS-* back/
# sudo tee CentOS-Linux-BaseOS.repo<<EOM
[baseos]
name=CentOS Linux \$releasever - BaseOS
baseurl=http://vault.centos.org/8.1.1911/BaseOS/\$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
EOM

# sudo tee CentOS-Linux-AppStream.repo<<EOM
[appstream]
name=CentOS Linux \$releasever - AppStream
baseurl=http://vault.centos.org/8.1.1911/AppStream/\$basearch/os/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial
EOM
# sudo cp CentOS-Linux-BaseOS.repo /root/CentOS-Linux-BaseOS.repo
# sudo cp CentOS-Linux-AppStream.repo /root/CentOS-Linux-AppStream.repo
# dnf makecache
Extra Packages for Enterprise Linux 8 - x86_64                4.7 MB/s |  13 MB     00:02    
CentOS-8 - Base                                               1.1 MB/s | 4.6 MB     00:04    
Extra Packages for Enterprise Linux Modular 8 - x86_64        197 kB/s | 733 kB     00:03    
CentOS-8 - AppStream                                          1.3 MB/s | 8.4 MB     00:06    
CentOS-8 - Extras                                             4.3 kB/s |  10 kB     00:02    
Module yaml error: Unexpected key in data
Module yaml error: Unexpected key in data
Module yaml error: Unexpected key in data
Module yaml error: Unexpected key in data
Module defaults error: Unexpected key in data
Module defaults error: Unexpected key in data
Module defaults error: Unexpected key in data
Module defaults error: Unexpected key in data
Metadata cache created.
```
#### 2.3 升级到CentOS 8
删除所有安装过的kernel:      

```
# rpm -e `rpm -q kernel`
```
删除`sysvinit-tools`等冲突包：     

```
# rpm -e --nodeps sysvinit-tools sysvinit-tools python36-rpmconf 
```
执行以下命令，升级到CentOS 8:      

```
# dnf -y --releasever=8 --allowerasing --setopt=deltarpm=false distro-sync
```
升级成功后，`/etc/yum.repos.d`目录下会被覆盖，回归到8.1版本

```
# cd /etc/yum.repos.d
# mkdir back1
# mv CentOS-* back1/
# sudo cp /root/CentOS-Linux-BaseOS.repo .
# sudo cp /root/CentOS-Linux-AppStream.repo .
```
安装内核包:       

```
# dnf makecache
# dnf install -y kernel-core
```
安装`minimal`及`Core`包组合:        

```
# mv /etc/yum/protected.d /etc/yum/protected.d.back
# dnf -y groupupdate "Core" "Minimal Install"
```
#### 2.4 升级CentOS8.5
回归原来的包配置:    

```
# cd /etc/yum.repos.d/
# rm -f CentOS-*
# mv back1/* .
# CENTOS_BASE=http://vault.centos.org
# sed -i -e "s|mirrorlist=|#mirrorlist=|g" /etc/yum.repos.d/CentOS-* 
# sed -i -e 's|#baseurl=http://mirror.centos.org|baseurl='$CENTOS_BASE'|g' /etc/yum.repos.d/CentOS-*
# dnf update
```
更新完毕后，重启，而后检查内核及操作系统版本:    

```
# uname -a
Linux localhost.localdomain 4.18.0-348.7.1.el8_5.x86_64 #1 SMP Wed Dec 22 13:25:12 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
# cat /etc/redhat-release 
CentOS Linux release 8.5.2111
```
### 3. CentOS 8 -> Rocky Linux 8
下载Rocky Linux升级脚本:     

```
# dnf -y install wget
# wget https://raw.githubusercontent.com/rocky-linux/rocky-tools/main/migrate2rocky/migrate2rocky.sh
# chmod a+x migrate2rocky.sh
```
升级到Rocky Linux 8.6:      

```
./migrate2rocky.sh -r
```

![/images/2022_11_02_16_04_40_865x689.jpg](/images/2022_11_02_16_04_40_865x689.jpg)

等待一段时间(取决于网速及机器配置，vagrant虚拟机约15分钟)，等待升级完成:    

![/images/2022_11_02_16_11_03_707x457.jpg](/images/2022_11_02_16_11_03_707x457.jpg)

### 4. 验证
重启该机器后，验证升级情况:     

```
[vagrant@localhost ~]$ uname -a
Linux localhost.localdomain 4.18.0-372.32.1.el8_6.x86_64 #1 SMP Thu Oct 27 15:18:36 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
[vagrant@localhost ~]$ cat /etc/os-release
NAME="Rocky Linux"
VERSION="8.6 (Green Obsidian)"
ID="rocky"
ID_LIKE="rhel centos fedora"
VERSION_ID="8.6"
PLATFORM_ID="platform:el8"
PRETTY_NAME="Rocky Linux 8.6 (Green Obsidian)"
ANSI_COLOR="0;32"
CPE_NAME="cpe:/o:rocky:rocky:8:GA"
HOME_URL="https://rockylinux.org/"
BUG_REPORT_URL="https://bugs.rockylinux.org/"
ROCKY_SUPPORT_PRODUCT="Rocky Linux"
ROCKY_SUPPORT_PRODUCT_VERSION="8"
REDHAT_SUPPORT_PRODUCT="Rocky Linux"
REDHAT_SUPPORT_PRODUCT_VERSION="8"
```
