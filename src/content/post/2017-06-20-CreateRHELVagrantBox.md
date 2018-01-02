+++
categories = ["Linux"]
keywords = ["Linux"]
date = "2017-06-20T14:41:41+08:00"
description = "Tips on create vagrant box on RHEL"
title = "创建RHELVagrantBox"

+++
### 背景
调研RHEL6.5, 为了在vagrant环境中验证我们的修改，故创建该系统的vagrant box
### 准备
Virtualbox 5.1.22 r115126, vagrant 1.9.1, CentOS 7.3(host机器)    
创建一台虚拟机，配置如下:    

```
内存: 512 m
网卡: NAT, port forward: 2223 -> 22
硬盘: 40 G
声卡: 禁用
```
用RHEL 6.5的ISO安装系统，安装完毕之后，将自动重启。     

### 配置
激活网络，通过配置`/etc/sysconfig/network-scripts/ifcfg-eth0`,
设置为`boot=yes`.    

安装完毕后，依然插入RHEL 6.5 ISO, 将其挂载到/mnt目录，并配置本地安装源:    

```
# mount /dev/sr0 /mnt
# vim /etc/yum.repos.d/local.repo
[local]
name=local
baseurl=file:///mnt
enabled=1
gpgcheck=0
# yum makecache&&yum install -y vim kernel-devel gcc bzip2
```
添加vagrant用户:    

```
# useradd -m vagrant
# passwd vagrant
```
添加`vagrant`用户到visudo:    

```
# visudo 
vagrant ALL=(ALL)	NOPASSWD:ALL
Defaults:vagrant	!requiretty
```

if you have make vagrant to be the member of `%wheel`, be sure to add
nopassword for this group.    

预置ssh-key:    

```
# mkdir -p /home/vagrant/.ssh
# chmod 0700 /home/vagrant/.ssh
# wget --no-check-certificate \
    https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub \
    -O /home/vagrant/.ssh/authorized_keys
# chmod 0600 /home/vagrant/.ssh/authorized_keys
# chown -R vagrant /home/vagrant/.ssh
```
配置ssh登录:     

```
# vim /etc/ssh/sshd_config
AuthorizedKeysFile .ssh/authorized_keys
```

### VBoxAdditional iso
在虚拟机的界面上点击`Device -> Install Guest Additional CD image`， 而后:    

```
# mount /dev/sr0 /mnt
# cd /mnt
# ./VBoxLinuxAdditions.run
```
### 压缩
使用dde命令清除空余空间:    

```
# dd if=/dev/zero of=/EMPTY bs=1M && rm -f /EMPTY
# shutdown -h now
```
形成rhel vagrant包:    

```
# vagrant package --base rhelbox
```

### 测试
安装镜像文件:    

```
# vagrant box add package.box --name "rhel65"
# vagrant init rhel65
# vagrant up
```

### CentOS7
First you have to copy `/etc/sysconfig/network-scripts/ifcfg-enp0sxxx` to
`/etc/sysconfig/network-scripts/ifcfg-eth0`, and edit its configuration by
replacing `enp0sx` to `eth0`, then you have to edit the grub configuration for
naming from `enp0sx` to `eth0`:    

```
Edit your /etc/default/grub. Change the line from
GRUB_CMDLINE_LINUX=""
to
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
```
Finally you make the new configuration for grub2:    

```
# grub2-mkconfig -o /boot/grub2/grub.cfg
```

if you have make vagrant to be the member of `%wheel`, be sure to add
nopassword for this group.    
