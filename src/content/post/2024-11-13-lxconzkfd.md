+++
title= "lxconzkfd"
date = "2024-11-13T14:01:08+08:00"
description = "lxconzkfd"
keywords = ["Technology"]
categories = ["Technology"]
+++
Steps:    

```
apt install -y lxc lxcfs
reboot
cp /usr/share/lxc/config/common.conf /usr/share/lxc/config/common.conf.back
cp common.conf /usr/share/lxc/config/common.conf
lxc-create -t local -n  zkfdlxc -- -m /root/meta.tar.xz -f /root/zkfdlxc1.tar.xz
vim /var/lib/lxc/zkfdlxc/config
```
Added:    

```
lxc.mount.entry = /dev/fb0 dev/fb0 none bind,optional,create=file
lxc.mount.entry = /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry = /dev/dri/renderD128 dev/renderD128 none bind,optional,create=file
### allow tty8
lxc.mount.entry = /dev/tty7 dev/tty7 none bind,optional,create=file
lxc.mount.entry = /dev/tty8 dev/tty8 none bind,optional,create=file
lxc.mount.entry = /dev/tty0 dev/tty0 none bind,optional,create=file
#lxc.mount.entry = /dev/tty1 dev/tty1 none bind,optional,create=file
#lxc.mount.entry = /dev/tty2 dev/tty2 none bind,optional,create=file
#lxc.mount.entry = /dev/tty3 dev/tty3 none bind,optional,create=file
### allow all of the input
lxc.mount.entry = /dev/input dev/input none bind,optional,create=dir
### allow all of the snd
lxc.mount.entry = /dev/snd dev/snd none bind,optional,create=dir
```
Start:    

```
chmod 777 /dev/tty* && chmod 777 /dev/fb0  && chmod 777 /dev/dri/*
lxc-ls -f
lxc-start -n zkfdlxc
```
