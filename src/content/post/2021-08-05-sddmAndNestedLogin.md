+++
title= "sddmAndNestedLogin"
date = "2021-08-05T17:03:39+08:00"
description = "sddmAndNestedLogin"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 目标
系统化说明如何在vdi设备和idv设备上开启sddm的nested模式下Multiseat登录支持。

### 环境准备
vdi设备信息如下：    

```
Intel(R) Celeron(R) CPU  J1900  @ 1.99GHz
# free -m
              总计         已用        空闲      共享    缓冲/缓存    可用
内存：        3826         487        1676         317        1662        2758
交换：           0           0           0
# df -h
文件系统        容量  已用  可用 已用% 挂载点
udev            1.9G     0  1.9G    0% /dev
tmpfs           383M  872K  382M    1% /run
/dev/sda5        27G  5.1G   21G   20% /
# cat /etc/issue
Ubuntu 18.04.5 LTS \n \l
#  uname -a
Linux xxxx 5.3.0-24-generic #26~18.04.2-Ubuntu SMP Tue Nov 26 12:34:22 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```
安装必要的包:    

```
# sudo apt-get update -y
# sudo apt-get upgrade -y
# sudo apt-get install -y sddm xfce4 unzip autoconf automake libtool pkg-config build-essential
# sudo apt-get install -y x11proto-dev xserver-xorg-dev libxcb-util-dev libxcb-icccm4-dev libxcb-image0-dev libxcb-shm0-dev libxcb-randr0-dev vim cmake cmake-extras extra-cmake-modules libpam-dev libxcb-xkb-dev qt5-default libqt5qml5 qt5-qmltooling-plugins qtdeclarative5-dev qttools5-dev xutils-dev
```
### 编译相关包
编译`xf86-video-nested`:    

```
$ git clone https://github.com/smemc/xf86-video-nested.git
$ cd xf86-video-nested
$ ./autogen.sh
$ ./configure --prefix=/usr
$ make -j2
$ sudo make install
```
编译`sddm-nested`:    

```
$ git clone https://github.com/purplepalmdash/sddm-nested-multiseat.git
$ cd sddm-nested
$ mkdir build && cd build
$ cmake -DCMAKE_INSTALL_PREFIX=/usr       -DCMAKE_BUILD_TYPE=Release        -Wno-dev ..
$ make -j2
$ sudo make install
$ sudo install -v -dm755 -o sddm -g sddm /var/lib/sddm
$ sddm --example-config > sddm.example.conf
$ sudo cp -v  sddm.example.conf /etc/sddm.conf
```
生成配置文件:   

```
cat > /etc/pam.d/sddm << "EOF" &&
# Begin /etc/pam.d/sddm

auth     requisite      pam_nologin.so
auth     required       pam_env.so

auth     required       pam_succeed_if.so uid >= 1000 quiet
auth     include        common-auth

account  include        common-account
password include        common-password

session  required       pam_limits.so
session  include        common-session

# End /etc/pam.d/sddm
EOF

cat > /etc/pam.d/sddm-autologin << "EOF" &&
# Begin /etc/pam.d/sddm-autologin

auth     requisite      pam_nologin.so
auth     required       pam_env.so

auth     required       pam_succeed_if.so uid >= 1000 quiet
auth     required       pam_permit.so

account  include        common-account

password required       pam_deny.so

session  required       pam_limits.so
session  include        common-session

# End /etc/pam.d/sddm-autologin
EOF

cat > /etc/pam.d/sddm-greeter << "EOF"
# Begin /etc/pam.d/sddm-greeter

auth     required       pam_env.so
auth     required       pam_permit.so

account  required       pam_permit.so
password required       pam_deny.so
session  required       pam_unix.so
-session optional       pam_systemd.so

# End /etc/pam.d/sddm-greeter
EOF
```
### 配置分屏
定义出Intel显卡的输出口：    

```
# vim  /etc/X11/xorg.conf.d/20-intel.conf 
Section "Device"
   Identifier  "Intel Graphics"
   Driver      "intel"
   Option      "AccelMethod"  "sna"
   Option      "TearFree"    "true"
   Option      "DRI"    "3"
   Option      "Monitor-VGA" "VGA"
   Option      "Monitor-HDMI2" "HDMI2"
EndSection
```
定义出显示器配置:    

```
Section "Monitor"
	Identifier	"VGA"
EndSection

Section "Monitor"
	Identifier	"HDMI2"
	Option		"Position"	"1920 0"
	Option		"PreferredMode"	"1920x1080"
EndSection

Section "Screen"
	Identifier	"VGA"
	Monitor		"VGA"
	SubSection "Display"
		Depth	24
		Modes	"1920x1080"
	EndSubSection
EndSection

Section "Screen"
	Identifier	"HDMI2"
	Monitor		"HDMI2"
	SubSection "Display"
		Depth	24
		Modes	"1920x1080"
	EndSubSection
EndSection

Section "ServerLayout"
	Identifier	"L1"
	Screen		"VGA"	0 0
	Screen		"HDMI2"		1920 0
	Option		"BlankTime"	"0"
	Option		"StandbyTime"	"0"
	Option		"SuspendTime"	"0"
	Option		"OffTime"	"0"
EndSection


Section "ServerFlags"
    Option        "BlankTime"    "0"
    Option        "StandbyTime"    "0"
    Option        "SuspendTime"    "0"
    Option        "OffTime"    "0"
EndSection

Section "Extensions"
    Option        "DPMS"    "Disable"
EndSection
```
定义出`seat1`和`seat2`的配置:    

```
# vim /etc/X11/seat1.conf 
Section "Module"
    Load        "shadow"
    Load        "fb"
EndSection

Section "Device"
    Identifier    "seat1"
    Driver        "nested"
EndSection

Section "Screen"
    Identifier    "Screen1"
    Device        "seat1"
    DefaultDepth    24
    SubSection "Display"
        Depth 24
        #Modes "3840x2160"
        Modes "1920x1080"
    EndSubSection
    Option        "Origin"    "1920 0"
EndSection

Section "ServerLayout"
    Identifier    "Nested"
    Screen        "Screen1"
    Option        "AllowEmptyInput"    "true"
    Option        "BlankTime"    "0"
    Option        "StandbyTime"    "0"
    Option        "SuspendTime"    "0"
    Option        "OffTime"    "0"
EndSection

Section "ServerFlags"
        Option "BlankTime" "0"
        Option "StandbyTime" "0"
        Option "SuspendTime" "0"
        Option "OffTime" "0"
EndSection
# vim /etc/X11/seat2.conf 
Section "Module"
    Load        "shadow"
    Load        "fb"
EndSection

Section "Device"
    Identifier    "seat2"
    Driver        "nested"
EndSection

Section "Screen"
    Identifier    "Screen1"
    Device        "seat2"
    DefaultDepth    24
    SubSection "Display"
        Depth 24
        Modes "1920x1080"
    EndSubSection
    Option        "Origin"    "0 0"
EndSection

Section "ServerLayout"
    Identifier    "Nested"
    Screen        "Screen1"
    Option        "AllowEmptyInput"    "true"
    Option        "BlankTime"    "0"
    Option        "StandbyTime"    "0"
    Option        "SuspendTime"    "0"
    Option        "OffTime"    "0"
EndSection

Section "ServerFlags"
        Option "BlankTime" "0"
        Option "StandbyTime" "0"
        Option "SuspendTime" "0"
        Option "OffTime" "0"
EndSection
```

定义出USB口与seat的映射关系(注意前置的口USB编号与机箱后部的USB编号的对应关系):    

```
# vim /etc/udev/rules.d/70-seat.rules 
SUBSYSTEM=="input", DEVPATH=="/devices/pci0000:00/0000:00:14.0/usb1/1-1/*", TAG+="seat",  ENV{ID_SEAT}="seat2", TAG+="master-of-seat", PROGRAM="/usr/bin/sed -n 's/.*startseat=\([^ ]*\).*/\1/p' /proc/cmdline", RESULT=="true"
SUBSYSTEM=="input", DEVPATH=="/devices/pci0000:00/0000:00:14.0/usb1/1-4/*", TAG+="seat",  ENV{ID_SEAT}="seat2", TAG+="master-of-seat", PROGRAM="/usr/bin/sed -n 's/.*startseat=\([^ ]*\).*/\1/p' /proc/cmdline", RESULT=="true"
SUBSYSTEM=="input", ATTRS{name}=="Power Button", TAG+="seat",  ENV{ID_SEAT}="seat2", TAG+="master-of-seat", PROGRAM="/usr/bin/sed -n 's/.*startseat=\([^ ]*\).*/\1/p' /proc/cmdline", RESULT=="true"
#SUBSYSTEM=="input", DEVPATH=="/devices/pci0000:00/0000:00:14.0/usb1/1-9/*", TAG+="seat",  ENV{ID_SEAT}="seat1", TAG+="master-of-seat", PROGRAM="/usr/bin/sed -n 's/.*startseat=\([^ ]*\).*/\1/p' /proc/cmdline", RESULT=="true"

SUBSYSTEM=="input", DEVPATH=="/devices/pci0000:00/0000:00:14.0/usb1/1-2/*", TAG+="seat",  ENV{ID_SEAT}="seat1", TAG+="master-of-seat", PROGRAM="/usr/bin/sed -n 's/.*startseat=\([^ ]*\).*/\1/p' /proc/cmdline", RESULT=="true"
#SUBSYSTEM=="input", DEVPATH=="/devices/pci0000:00/0000:00:14.0/usb1/1-6/*", TAG+="seat",  ENV{ID_SEAT}="seat1", TAG+="master-of-seat", PROGRAM="/usr/bin/sed -n 's/.*startseat=\([^ ]*\).*/\1/p' /proc/cmdline", RESULT=="true"
SUBSYSTEM=="input", ATTRS{name}=="Sleep Button", TAG+="seat",  ENV{ID_SEAT}="seat1", TAG+="master-of-seat", PROGRAM="/usr/bin/sed -n 's/.*startseat=\([^ ]*\).*/\1/p' /proc/cmdline", RESULT=="true"
```
开启multi-seat:    

```
# vim /etc/default/grub
.....
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash startseat=true"
GRUB_CMDLINE_LINUX="startseat=true"
....
# update-grub2
```
重启后验证:    

```
# cat /proc/cmdline 
BOOT_IMAGE=/boot/vmlinuz-5.3.0-24-generic root=UUID=50368a13-e1d7-4699-8b3b-b5ffc1d148e6 ro startseat=true quiet splash startseat=true vt.handoff=1
```
### 配置多用户登录
创建两个用户`seat1`和`seat2`:    

```
# useradd -m seat1
# useradd -m seat2
# passwd seat1
# passwd seat2
```
配置其自动登录(`[AutoLogin]`内只保留如下所示部分):    

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
```
### Awesome自启动项
在`awesome`的情景下，我们配置其登录方式为:    

```
# su seat1
$ /bin/bash
$ mkdir -p ~/.config/awesome
$ cp /etc/xdg/awesome/rc.lua ~//.config/awesome
$ vim ~/.config/awesome/autorun.sh

$ vim ~/.config/awesome/autorun.sh 
    #!/usr/bin/env bash
    
    function run {
      if ! pgrep -f $1 ;
      then
        $@&
      fi
    }
    run sleep3 && /opt/ctg/CtyunClouddeskUniversal/CtyunClouddeskUniversal
$ vim ~/.config/awesome/rc.lua 
最后一行添加:
awful.spawn.with_shell("~/.config/awesome/autorun.sh")
$ chmod 777 ~/.config/awesome/autorun.sh 
$ rm -rf /tmp/awesome/
$ cp -r ~/.config/awesome /tmp
$ exit
exit
$ exit
# su seat2
$ /bin/bash
$ cp -r /tmp/awesome/ ~/.config/
$ chmod 777 ~/.config/awesome/autorun.sh 
$ exit
exit
$ exit
```
现在重启后即可看到结果，其结果表现为，两路显示独立输出且进入到idv登录前界面。   
