+++
title= "WorkingTipsOnLXDccse"
date = "2021-05-09T07:13:06+08:00"
description = "WorkingTipsOnLXDccse"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 目标
在LXD上运行ccse
### 准备
服务器上安装lxd, 初始化镜像centos7, ccse安装介质。  

### 步骤
创建一个profile， 用于创建lxd用于部署验证:    

```
lxc profile show default>ccse
vim ccse 
lxc profile create ccse
lxc profile edit ccse<ccse
```
文件的内容如下：    

```
config:
  linux.kernel_modules: ip_tables,ip6_tables,netlink_diag,nf_nat,overlay,br_netfilter,xt_conntrack
  raw.lxc: "lxc.apparmor.profile=unconfined\nlxc.cap.drop= \nlxc.cgroup.devices.allow=a\nlxc.mount.auto=p
roc:rw sys:rw"
  security.nesting: "true"
  security.privileged: "true"
description: CCSE Running profile
devices:
  eth0:
    name: eth0
    network: lxdbr0
    type: nic
  hashsize:
    path: /sys/module/nf_conntrack/parameters/hashsize
    source: /dev/null
    type: disk
  kmsg:
    path: /dev/kmsg
    source: /dev/kmsg
    type: unix-char
  root:
    path: /
    pool: ssd
    type: disk
name: ccse
```
验证此profile是否可正常工作:    

```
# lxc launch centos7 kkk --profile ccse
Creating kkk
Starting kkk                              
# lxc exec kkk bash
[root@kkk ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
```
注意：
1. 版本略高于推荐的centos 7.6. 
2. 使用上述的权限文件加载，可以解决teledb组碰到的获取磁盘权限问题。    

### 部署介质准备
初始化容器:    

```
cd /etc/yum.repos.d/
mkdir back
mv * back
vi ccse.repo
yum makecache
vi /etc/yum.conf 
yum install -y which vim net-tools lsof sudo
```
因为需要将lxd当成物理机来使用，安装openssh-server后重启：    

```
 yum install -y openssh-server
 systemctl enable sshd
 systemctl start sshd
 passwd
 reboot
```
再次进入容器后，下载安装文件:    

```
scp docker@xxx.xxx.xxx.xx:/home/docker/shrink280/ccse-installer-2.8.0-rc-linux-amd64-offline-20210409204619-shrink.tar.xz .
tar xJf ccse-installer-2.8.0-rc-linux-amd64-offline-20210409204619-shrink.tar.xz
```
### 部署console节点
记录ip 地址` 10.222.125.68`， 配置完正确的IP地址后，按原有步骤安装console节点，安装完毕后上传镜像。   

### 制作基础节点
需打包节点所需要的依赖：    

```
# lxc launch centos7 base
# lxc exec base bash
     yum install -y which lsof vim net-tools sudo selinux-policy libseccomp libselinux-python selinux-policy-targeted openssh-server ebtables ethtool
     systemctl enable sshd
     passwd
     shutdown -h now
#  lxc publish base --alias ccsenode

```


hashsize:    

```
sudo su
echo "262144" > /sys/module/nf_conntrack/parameters/hashsize
cat /sys/module/nf_conntrack/parameters/hashsize
```
