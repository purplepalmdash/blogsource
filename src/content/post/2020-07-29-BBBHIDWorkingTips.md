+++
title= "BBBHIDWorkingTips"
date = "2020-07-29T15:35:12+08:00"
description = "BBBHIDWorkingTips"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Preparation
The system is:   

```
debian@beaglebone:~$ cat /etc/issue
Debian GNU/Linux 9 \n \l
BeagleBoard.org Debian Stretch imgtec Image 2020-04-06
Support: http://elinux.org/Beagleboard:BeagleBoneBlack_Debian
default username:password is [debian:temppwd]
debian@beaglebone:~$ uname -a
Linux beaglebone 4.14.108-ti-r131 #1stretch SMP PREEMPT Tue Mar 24 19:18:37 UTC 2020 armv7l GNU/Linux
```
Change password via `passwd`.    
Enlarge the partition from 4GB to 32GB(32 GB disk)    

```
root@beaglebone:/home/debian# /opt/scripts/tools/grow_partition.sh 
# vim /etc/network/interfaces
auto eth0
iface eth0 inet dhcp
# reboot
##### Checking #############
debian@beaglebone:~$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            217M     0  217M   0% /dev
tmpfs            49M  5.2M   44M  11% /run
/dev/mmcblk0p1   30G  2.5G   26G   9% /
```
Edit the sources.list configuration:    

```
$ cat /etc/apt/sources.list
deb http://mirrors.ustc.edu.cn/debian stretch main contrib non-free
#deb-src http://mirrors.ustc.edu.cn/debian stretch main contrib non-free

deb http://mirrors.ustc.edu.cn/debian stretch-updates main contrib non-free
#deb-src http://mirrors.ustc.edu.cn/debian stretch-updates main contrib non-free

deb http://mirrors.ustc.edu.cn/debian-security stretch/updates main contrib non-free
#deb-src http://mirrors.ustc.edu.cn/debian-security stretch/updates main contrib non-free
#  apt-get update -y 
# apt-get install -y libudev-dev libusb-dev awesome iotop
```

### USB HID

