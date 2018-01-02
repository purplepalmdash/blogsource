+++
keywords = ["Linux"]
description = "Linux Encryption"
title = "Linux下的磁盘加密"
date = "2017-01-19T10:26:41+08:00"
categories = ["Linux"]

+++
### 目的
调研Linux虚拟机磁盘加密

### 环境
Ubuntu16.04 LTS/Libvirt.    

### 全盘加密
选择磁盘全盘加密，需要在系统安装时指定encrypted LVM:    

![/images/2017_01_19_11_16_10_884x316.jpg](/images/2017_01_19_11_16_10_884x316.jpg)    

指定加密的密码:    

![/images/2017_01_19_11_58_50_471x231.jpg](/images/2017_01_19_11_58_50_471x231.jpg)    

### 效果
每次启动时，需要在以下界面手动输入加密时密码，以继续系统启动:    

![/images/2017_01_19_12_06_36_721x145.jpg](/images/2017_01_19_12_06_36_721x145.jpg)    

如果输入失败，会提示密码错误，无法继续启动:    

![/images/2017_01_19_12_10_23_517x99.jpg](/images/2017_01_19_12_10_23_517x99.jpg)    

只有输入正确密码，才可以继续系统启动并进入到登录界面:    

![/images/2017_01_19_12_12_35_338x141.jpg](/images/2017_01_19_12_12_35_338x141.jpg)    

### Packer.io制作加密系统
UBUNTU系统，对于加密LVM系统的构建，需要改动相应的preseed文件。    
CentOS需要改动KickStart中对应的定义.    

### 磁盘格局
通过命令`sudo fdisk -l`查看磁盘格局:    

```
Disk /dev/ram0 ~ /dev/ram14, 64 MiB, 67108864 bytes, 131072 sectors
Disk /dev/mapper/sda5_crypt: 7.5 GiB, 8074035200 bytes, 15769600 sectors
Disk /dev/mapper/ubuntu--vg-root: 6.5 GiB, 6996099072 bytes, 13664256 sectors
Disk /dev/mapper/ubuntu--vg-swap_1: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk /dev/mapper/cryptswap1: 1023.5 MiB, 1073217536 bytes, 2096128 sectors
```
通过`df -h`查看分区格式:    

```
$ df -h
Filesystem                   Size  Used Avail Use% Mounted on
udev                         478M     0  478M   0% /dev
tmpfs                        100M  3.3M   96M   4% /run
/dev/mapper/ubuntu--vg-root  6.3G  1.6G  4.4G  27% /
tmpfs                        497M  4.0K  497M   1% /dev/shm
tmpfs                        5.0M     0  5.0M   0% /run/lock
tmpfs                        497M     0  497M   0% /sys/fs/cgroup
/dev/sda1                    472M  102M  346M  23% /boot
tmpfs                        100M     0  100M   0% /run/user/1000
/home/xxx/.Private          6.3G  1.6G  4.4G  27% /home/xxx
```
.Private分区内容：    

```
$ pwd
/home/xxxx/.Private
$ ls
ECRYPTFS_FNEK_ENCRYPTED.FWYRZgbFHyb3vETJehOMeGaNdKMCGZhYJy6B-0l.TGAjZjnshofl6TMQF---
ECRYPTFS_FNEK_ENCRYPTED.FWYRZgbFHyb3vETJehOMeGaNdKMCGZhYJy6Bdpm6RnbmADLIif6RGU1S5k--
ECRYPTFS_FNEK_ENCRYPTED.FWYRZgbFHyb3vETJehOMeGaNdKMCGZhYJy6BLUWJcoDqnHFsQ4A3epMc-E--
ECRYPTFS_FNEK_ENCRYPTED.FWYRZgbFHyb3vETJehOMeGaNdKMCGZhYJy6BS7V4kKNhThl7lqZKTP5ESk--
ECRYPTFS_FNEK_ENCRYPTED.FWYRZgbFHyb3vETJehOMeGaNdKMCGZhYJy6Bw8q1hT0peM0rGRaQYYWDzE--
ECRYPTFS_FNEK_ENCRYPTED.FWYRZgbFHyb3vETJehOMeGaNdKMCGZhYJy6BXMsQUOdEgj0L-co6klbTaE--
ECRYPTFS_FNEK_ENCRYPTED.FWYRZgbFHyb3vETJehOMeGaNdKMCGZhYJy6BZN3TyfKVLDHINFE1.88qzE--
ECRYPTFS_FNEK_ENCRYPTED.FWYRZgbFHyb3vETJehOMeGaNdKMCGZhYJy6BZUKbEf0KxQzl84GnQZl73E--
ECRYPTFS_FNEK_ENCRYPTED.FXYRZgbFHyb3vETJehOMeGaNdKMCGZhYJy6BV-b9978jYgqh05Dadlf8vsKlFwiW7mAteHwsvaUddlc-
```

### 用户目录加密
安装时，指定以下参数:    

![/images/2017_01_19_15_22_41_739x214.jpg](/images/2017_01_19_15_22_41_739x214.jpg)    

指定磁盘分区格式为基本格式(非LVM）。    

检查磁盘分区:    

```
$ df -h
Filesystem           Size  Used Avail Use% Mounted on
udev                 478M     0  478M   0% /dev
tmpfs                100M  3.3M   96M   4% /run
/dev/sda1            6.8G  1.1G  5.4G  17% /
tmpfs                497M  4.0K  497M   1% /dev/shm
tmpfs                5.0M     0  5.0M   0% /run/lock
tmpfs                497M     0  497M   0% /sys/fs/cgroup
tmpfs                100M     0  100M   0% /run/user/1000
/home/dash/.Private  6.8G  1.1G  5.4G  17% /home/dash
tmpfs                100M     0  100M   0% /run/user/1001
```

