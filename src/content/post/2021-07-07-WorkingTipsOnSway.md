+++
title= "WorkingTipsOnSway"
date = "2021-07-07T08:31:25+08:00"
description = "WorkingTipsOnSway"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install sway via:    

```
$ sudo pacman -S sway alacritty
```
Logout and type `sway` in terminal, here we encounter Nvidia issue:   

```
$ sway
sway/main.c:100Proprietary Nvidia driver are NOT supported. Use Nouveau. To launch ....
$ sway --my-next-gpu-wont-be-nvidia
Could not connect to socket /run/seated.sock: No such file or directory
$ echo LIBSEAT_BACKEND=logind >> /etc/environment 
$ sway --my-next-gpu-wont-be-nvidia 2>&1 | tee start.log
sway/main.c: 202 unable to drop root
$ sudo useradd -m dash1
$ sudo passwd dash1
$ exit
Login with the newly created dash1 and re-test
```

Install gdm and change to opensource nvidia driver:    

```
$ sudo pacman -S gdm
$ sudo rm -f /etc/modprobe.d/nouveau_blacklist.conf
$ sudo vim /etc/mkinitcpio.conf
MODULES=(... nouveau ...)

$ sudo  mkinitcpio -p linux
```
