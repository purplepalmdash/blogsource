+++
title = "TroubleShootingOnVMWareBasedVM"
date = "2019-02-19T08:29:42+08:00"
description = "TroubleShootingOnVMWareBasedVM"
keywords = ["Linux"]
categories = ["Linux"]
+++
记录一下昨天遇到的一个问题：    
同事的虚拟机(Ubuntu)自检失败，fsck总是不过，直接抛到initramfs的界面。    

试图探索:    
启动的时候按Shift键，到grub界面，以recovery模式启动，依然失败。看来是ext4的磁盘inode有错误。    

解决方案：    
用livecd启动系统，启动以后到系统里去fsck修复该文件系统。   
gparted启动失败。    
后面我们用的是一个CentOS7的livecd系统进入的系统,     

```
# fsck -y /dev/mapper/lv_lv_root
```
扫描并自动修复(yes)磁盘上的inode错误。修复以后，用硬盘启动系统，可正常进入系统。避免了重新安装操作系统的开销。    
