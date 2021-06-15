+++
title= "iPadiPhoneMirroringDisplay"
date = "2021-06-15T06:30:43+08:00"
description = "iPadiPhoneMirroringDisplay"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. 硬件/配置
用来作为AirPlay Mirror Server的设备是一个brix投影电脑，配置为：    

```
Intel(R) Core(TM) i3-4010U CPU @ 1.70GHz
16 GB Ram
120 GB msata ssd
1000 M ethernet
```
安装操作系统为`ArchLinux`.     
###  2. 软件/安装配置步骤
通过`yay`安装`uxplay`:   

```
yay uxplay
```
使能服务:    

```
systemctl start avahi-daemon
systemctl enable avahi-daemon
```
此时最好重启，因uxplay安装过程中有涉及到其他模块。重启后可以通过以下命令开启`uxplay`, 此时IOS的屏幕镜像中找到`uxplay`的条目。    

```
$ sudo uxplay &
```

### 3. 自动Serving
上述的场景针对个人使用一般情况下是够了，但是如果使用者是家人，则上述的步骤需要改进。    

首先我们要让ArchLinux自动登录，自动登录后进入到图形界面。    
然后我们要用uxplay随桌面启动而启动。     
启动后我们需要禁用电源管理，以使得在长时间无键盘鼠标输入后屏幕依然可以保持常亮。    
针对IOS普遍存在的设备上可用空间较少的缘故，我们需要开启brix盒子上的samba服务，让IOS可以使用外置存储用来播放视频。     
IOS本身需要安装支持SAMBA的播放软件。     

分而击之，一一解决这些问题。      

#### 3.1 自动登录
以用户`dash`为例，说明如何实现`lightdm+awesome`的自动登录过程：   

```
$ sudo pacman -S lightdm lxde
# pacman -S xorg lightdm-gtk-greeter xterm xorg-xinit awesome
# systemctl enable lxdm.service
# vim /etc/lxdm/lxdm.conf
####autologin=dgod
autologin=dash
# vim /home/dash/.dmrc
[Desktop]
Session=awesome
# groupadd -r autologin
# gpasswd -a dash autologin
# vim /etc/lightdm.conf
[LightDM]
在此条目中做如下修改：
user-session = awesome
autologin-user = dash
autologin-session = awesome
[Seat:*]
在此条目中做如下修改：   
pam-service=lightdm
pam-autologin-service=lightdm-autologin
greeter-session=lightdm-gtk-greeter
user-session=awesome
session-wrapper=/etc/lightdm/Xsession
autologin-user=dash
autologin-user-timeout=0
```
#### 3.2 uxplay自启动
将`sudo uxplay &` 添加到~/.config/awesome/rc.lua条目中即可，具体步骤参见`awesome`的一般配置文档。需要注意的是`dash`用户需要免密码使用`sudo `  .    
#### 3.3 禁用屏幕电源管理
将`sudo xset -dpms &` 添加到~/.config/awesome/rc.lua条目中即可.    
#### 3.4  samba服务
参考ArchLinux Wiki条目
#### 3.5 IOS samba player
软件市场安装vlc, vlc中内建对samba Server的流媒体播放支持。    
