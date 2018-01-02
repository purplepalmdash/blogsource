+++
categories = ["Linux"]
keywords = ["Linux"]
description = "Ubuntu Disk Encryption"
title = "Ubuntu磁盘加密测试"
date = "2017-02-08T16:25:34+08:00"

+++
### 准备
三台虚拟机，Ubuntu16.04 安装盘，virt-manager.    

### 角色配置
虚拟机A：未加密，Ubuntu16.04安装。    
虚拟机B: encrypted LVM, 安装Ubuntu16.04.    
虚拟机C：Test Machine.    

### 加密配置
每次启动时避免输入密码的配置， 在虚拟机B上作如下配置:    

首先备份initrd.img:    

```
# cp /boot/initrd.img-4.4.0-31-generic /boot/initrd.img-4.4.0-31-generic.safe
```
在未经加密的`/boot`分区生成加密的key文件:    

```
# dd if=/dev/urandom of=/boot/keyfile bs=1024 count=4
```
查看加密分区情况:    

```
$ sudo blkid | grep -i crypto
/dev/vda5: UUID="a255260b-30eb-4630-b9c9-a6b7f75b236e" TYPE="crypto_LUKS" PARTUUID="2a203ff6-05"
```
从上面可以看出vda5是我们的加密风趣，现在将新创建的keyfile作为加密卷的解锁文件:    

```
# cryptsetup -v luksAddKey /dev/vda5 /boot/keyfile 
Enter any passphrase:
```
输入你以前创建的密码，看到以下输出时代表解锁文件添加成功:    

```
Key slot 0 unlocked.
Command successful.
```
现在更改`/etc/crypttab`文件:    

```
# cp /etc/crypttab /root/
# vim /etc/crypttab
vda5_crypt UUID=a255260b-30eb-4630-b9c9-a6b7f75b236e /dev/disk/by-uuid/36747581-1841-47de-9ce2-b1262e1eb167:/keyfile luks,keyscript=/lib/cryptsetup/scripts/passdev
```
其中`/ev/disk/by-uuid`的字段可以通过`blkid`来获得，即`/boot`的uuid值。    
如果无法修改该文件，记得改变其权限，修改完毕后，更改回以下权限:    

```
# chmod 0440 /etc/crypttab
```
重新生成内核并启动:    

```
# mkinitramfs -o /boot/initrd.img-4.4.0-31-generic 4.4.0-31-generic
```

### 系统更新
测试一下系统更新对加密磁盘的影响, 无。
