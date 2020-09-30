+++
title= "JogglerOpenFrame"
date = "2020-09-27T11:14:17+08:00"
description = "JogglerOpenFrame"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Reflash latest firmware
Examine the disk and write the image to disk:   
```
Disk /dev/sdd：1.88 GiB，2021654528 字节，3948544 个扇区
磁盘型号：                
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x2fbbe9fe

设备       启动 起点    末尾    扇区  大小 Id 类型
/dev/sdd1  *       0 1390591 1390592  679M  0 空
/dev/sdd2        264  131335  131072   64M ef EFI (FAT-12/16/32)
dash@archnvme:/media/sda $ cd ~/Downloads 
dash@archnvme:~/Downloads $ sudo su   
[root@archnvme Downloads]# gzip -dc reflash111_of.img.gz | dd of=/dev/sdd bs=1M
```
Mount reflash's usb disk into computer:    

```
Mount the USB device on your system; you should see a volume named rfl-boot.
On the rfl-boot volume there is a directory named reflash.
Download the operating system image you would like to write, along with its MD5 file.
Copy both the compressed .img.gz file and its .img.gz.md5 counterpart into the reflash directory.
```
Now insert the flashed usb disk into joggler, then flash will begin.    

After flashing, could poweron from flash. 
