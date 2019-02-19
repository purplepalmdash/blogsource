+++
title = "TipsOnLightdm"
date = "2018-11-24T21:08:08+08:00"
description = "TipsOnLightdm"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Steps
Install lightdm via:    

```
$ sudo pacman -S lightdm
```
but this will cause gui startup failed.   

![/images/2018_11_24_21_09_12_541x252.jpg](/images/2018_11_24_21_09_12_541x252.jpg)

Changes to lxdm solves the problem:    

```
# pacman -S lxde
# systemctl enable lxdm.service
# vim /etc/lxdm/lxdm.conf
####autologin=dgod
# vim /root/.dmrc
[Desktop]
Session=xfce

```

Install:    

```
# pacman -S xorg lightdm-gtk-greeter xterm xorg-xinit awesome
# vim /etc/lightdm.conf
greeter-session=lightdm-gtk-greeter

Add configuration for lightdm-gtk-greeter
```
Desktop Manager session:    

```
 cat ~/.dmrc 
[Desktop]
Session=awesome
```

Install:    

```
$ groupadd -r autologin
$ gpasswd -a root autologin
$ pacman -S xfce4-goodies
$ pacman -S awesome
```

