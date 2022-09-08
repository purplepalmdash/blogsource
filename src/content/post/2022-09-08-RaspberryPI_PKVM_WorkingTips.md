+++
title= "RaspberryPI_PKVM_WorkingTips"
date = "2022-09-08T15:12:36+08:00"
description = "RaspberryPI_PKVM_WorkingTips"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Flash aosp 13 images
Image download URL:    
`https://konstakang.com/devices/rpi4/AOSP13/`   

Flash it into rpi 4b.    

### pkvm kernel/images
Download alpine kernel/images from(`https://www.worldofgh0st.com/blog/`):    

`https://www.worldofgh0st.com/wp-content/uploads/2022/09/KVM-Linux.zip`  

adb push them to `/data/local/tmp`.    

### Start crosvm machine
adb root and login with adb shell:    

```
➜  Downloads adb -s 192.168.1.114:5555 root 
restarting adbd as root
➜  Downloads adb -s 192.168.1.114:5555 shell
rpi4:/ # 
```
Start the crosvm vm and examine the vm info:   

```
rpi4:/ # /apex/com.android.virt/bin/crosvm run --disable-sandbox -p 'init=/bin/sh' --rwroot /data/local/tmp/alpine.img /data/local/tmp/kernel.img

..............

/ # uname -a
Linux (none) 5.19.0-13666-gffcf9c5700e4 #3 SMP PREEMPT Sat Sep 3 15:11:04 PDT 2022 aarch64 Linux
/ # cat /etc/issue
Welcome to Alpine Linux 3.16
Kernel \r on an \m (\l)

```

### Customize Kernel
In an arm64 server, clone the latest kernel and compile Image:    

```
# apt install build-essential flex bison libssl-dev -y
# git clone  git://mirrors.ustc.edu.cn/linux.git
# cd linux
# make defconfig
# make -j128
# ls arch/arm64/boot
dts  Image  Image.gz  install.sh  Makefile
```
Transfer the `Image` to rpi4's `/data/local/tmp/`    
### Customize Ubuntu20.04 rootfs
Get the latest aarch64 image and change the corresponding items:    

```
# axel https://cloud-images.ubuntu.com/releases/focal/release/ubuntu-20.04-server-cloudimg-arm64-root.tar.xz
# qemu-img create -f raw ubuntu.img 6G
# mkfs.ext4 ubuntu.img
# mount ubuntu.img /mnt
# tar xJvf ubuntu-20.04-server-cloudimg-arm64-root.tar.xz -C /mnt/
# cd /mnt
# touch etc/cloud/cloud-init.disabled
# vim etc/passwd
Change the first line :   
root::0:0:root:/root:/bin/bash
# umount /mnt
```
Transfer the `ubuntu.img` onto the rpi4's `/data/local/tmp/`     

### Start ubuntu vm
Start via:    

```
# /apex/com.android.virt/bin/crosvm run --disable-sandbox -p 'root=/dev/vda' --rwroot /data/local/tmp/ubuntu.img /data/local/tmp/Image
```

Examine the vm info:   

```
root@ubuntu:~# uname -a
Linux ubuntu 6.0.0-rc4-00062-g0066f1b0e275 #1 SMP PREEMPT Thu Sep 8 01:24:14 EDT 2022 aarch64 aarch64 aarch64 GNU/Linux
root@ubuntu:~# cat /etc/issue
Ubuntu 20.04.4 LTS \n \l

root@ubuntu:~# free -m
              total        used        free      shared  buff/cache   available
Mem:            221          88          17           0         115         127
Swap:             0           0           0
root@ubuntu:~# cat /proc/cpuinfo
processor	: 0
BogoMIPS	: 108.00
Features	: fp asimd evtstrm crc32 cpuid
CPU implementer	: 0x41
CPU architecture: 8
CPU variant	: 0x0
CPU part	: 0xd08
CPU revision	: 3
```
