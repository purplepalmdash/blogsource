+++
title= "WorkingTipsOnMultiSeatUbuntu200403"
date = "2021-11-10T17:03:17+08:00"
description = "WorkingTipsOnMultiSeatUbuntu200403"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 前提
基准操作系统为`ubuntu-20.04.3-live-server-amd64.iso`, 不可以安装为`desktop`版本的iso。 
### 系统及包更新
确保系统更新到最新:    

```
$ sudo apt-get update -y && sudo apt-get upgrade -y
```
安装必要的包(这里需要严格按照顺序来安装，否则会出现多安装包的情况导致`gdm`被安装后`sddm`无法正常工作):    

```
$ sudo apt-get install -y sddm unzip autoconf automake libtool pkg-config build-essential x11proto-dev xserver-xorg-dev libxcb-util-dev libxcb-icccm4-dev libxcb-image0-dev libxcb-shm0-dev libxcb-randr0-dev vim cmake cmake-extras extra-cmake-modules libpam-dev libxcb-xkb-dev qt5-default libqt5qml5 qt5-qmltooling-plugins qtdeclarative5-dev xutils-dev
$ sudo apt-get install openbox --no-install-recommends --no-install-suggests
$ sudo apt-get install -y xinit
```
### Multi-seat编译/配置
将`multi-seat-2004.tar.gz`上传到机器上，解压:    

```
$ tar xzvf multi-seat-2004.tar.gz
$ ls
multi-seat sddm
$ cd multi-seat/
$ ls
sddm_config  sddm-nested-multiseat  udev_config  xf86-video-nested  xorg_config
```
注意在20.04的系统中，不需要编译`sddm-nested-multiseat`，因为编译的过程中会引入`gdm3`，后期我们用直接拷贝二进制文件的方式安装`sddm-nested-multiseat`。 

编译`xf86-video-nested`包:    

```
$ cd xf86-video-nested
$ ./autogen.sh && ./configure --prefix=/usr && make -j2 && sudo make install
```
安装`sddm-nested`:    

```
$ cd sddm
$ ./install.sh
```
更新`sddm`登陆免密配置文件:   

```
$ cd sddm_config
$ sudo cp * /etc/pam.d
```
### 配置桌面登陆
更新`xorg`定义文件：  

```
$ cd xorg_config
$ sudo mkdir -p /etc/X11/xorg.conf.d
$ sudo cp xorg.conf.d/20-intel.conf /etc/X11/xorg.conf.d
$ sudo cp seat* /etc/X11
$ sudo cp /bin/sed /usr/bin/sed
```
配置USB口与seat的映射关系（从已有的例子进行修改）：    

```
$ cd udev_config
$ vim 70-seat.rules
修改完毕后：   
$ sudo cp 70-seat.rules /etc/udev/rules.d/70-seat.rules
```
开启multi-seat:    

```
$ sudo vim /etc/default/grub
.....
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash startseat=true"
GRUB_CMDLINE_LINUX="startseat=true"
....
$ sudo update-grub2
```
建立两个用于登陆的用户:    

```
$ sudo useradd -m seat1
$ sudo useradd -m seat2
$ sudo passwd seat1
$ sudo passwd seat2
```
配置`sddm`为两个用户的自动登陆(`[AutoLogin]`内只保留如下所示部分):    

```
# cat /etc/sddm.conf | more
[Autologin]
# Whether sddm should automatically log back into sessions when they exit
# Whether sddm should automatically log back into sessions when they exit
Relogin=false,false
SeatName=seat1,seat2
#Session=awesome,awesome
Session=xfce,xfce
User=seat1,seat2
....
此时重启后可以看到，双屏方案在登陆前自动进入到图形界面并已经实现分屏.   
```
