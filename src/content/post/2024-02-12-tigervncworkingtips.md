+++
title= "tigervncworkingtips"
date = "2024-02-12T23:40:25+08:00"
description = "tigervncworkingtips"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install lxqt for using for vnc:    

```
$ sudo pacman -S lxqt
```
Enable the vnc configration for using lxqt:    

```
$ cat ~/.vnc/config
session=lxqt
geometry=1920x1080
alwaysshared

```
enable the linux user for session:   

```
$ cat /etc/tigervnc/vncserver.users 
# TigerVNC User assignment
#
# This file assigns users to specific VNC display numbers.
# The syntax is <display>=<username>. E.g.:
#
# :2=andrew
# :3=lisa
 :1=dash
```
Now setup the vncpasswd:    

```
$ vncpasswd
```
Then you could start the vnc via:   

```
$ sudo systemctl start vncserver@:1
$ sudo systemctl enable vncserver@:1
```
