+++
title = "CreateRHEL6CustomizedISO"
date = "2017-07-27T12:28:35+08:00"
description = "CreateRHEL6CustomizedISO"
keywords = ["Linux"]
categories = ["Linux"]
+++

### 目的
根据用户自定义配置，自动从ISO安装出整个系统。     

### 准备材料
RHEL 6.6安装光盘, `x86_64`版本。
自定义kickstart文件，用于自定义分区/用户/密码/安装包等    
红帽系列操作系统(用于制作光盘镜像，已验证Redhat7.3)    

### 步骤
1. 创建目录用于挂载安装光盘和自定义光盘,
   其中`/media/bootiso`用于挂载安装光盘，
`/media/bootisoks`用于存放自定义光盘内容:    

```
$ mkdir -p /media/bootiso /media/bootisoks
```

2. 拷贝安装内容到自定义光盘目录:    

```
$ sudo mount -t iso9660 -o loop DVD.iso /media/bootiso
$ cp -r /media/bootiso/* /media/bootisoks/
$ chmdo -R u+w /media/bootisoks
$ cp /media/bootiso/.discinfo /media/bootisoks
$ cp /media/bootiso/.discinfo /media/bootisoks/isolinux
```

3. 拷贝自定义的ks文件到isolinux目录下:    

```
$ cp YourKickStartFile.ks /media/bootisoks/isolinux
```

4. 配置引导选项:    

```
$ vim /media/bootisoks/isolinux.cfg
initrd=initrd.img ks=cdrom:/isolinux/ks.cfg
```

5. 创建ISO文件:    

```
# mkisofs -r -T -V "MYISONAME" -b isolinux/isolinux.bin -c isolinux/boot.cat
-no-emul-boot -boot-load-size 4 -boot-info-table -o ../boot.iso .
```

经历此五个步骤以后，即可得到我们定制好的ISO，用此ISO即可安装出我们自定义好的系统.    

### kickstart示例文件:    
安装了基本桌面、中文支持等。    

```
#platform=x86, AMD64, or Intel EM64T
#version=DEVEL
# Firewall configuration
firewall --disabled
# Install OS instead of upgrade
install
# Use network installation
#url --url="http://10.7.7.2/CentOS"
cdrom
# Root password
rootpw --iscrypted xxxxxxxxxxxxxxxxxxxx
# System authorization information
auth  --useshadow  --passalgo=sha512
# Use graphical install
graphical
firstboot --disable
# System keyboard
keyboard us
# System language
lang en_US
# SELinux configuration
selinux --disabled
# Installation logging level
logging --level=info

# System timezone
timezone  Asia/Hong_Kong
# System bootloader configuration
bootloader --location=mbr
# Clear the Master Boot Record
zerombr
# Partition clearing information
clearpart --all  
# Disk partitioning information
part swap --fstype="swap" --size=1024
part / --asprimary --fstype="ext4" --grow --size=1

%packages
@basic-desktop
@chinese-support
@internet-browser
@x11
-ibus-table-cangjie
-ibus-table-erbi
-ibus-table-wubi

%end
```
其中`rootpw`以后的字段可以通过以下命令得到:    

```
$ openssl passwd -1 "Your_Password_Here"
```

### ks.cfg的另一种构建方法
在安装完的每一台机器上，都可以看到/root/ana...ks文件，编辑此文件即可得到我们定制化的kickstart配置。    
