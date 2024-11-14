+++
title= "lxconzkfdarm64"
date = "2024-11-14T09:16:54+08:00"
description = "lxconzkfdarm64"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install lxc:    

```
# apt install -y lxc lxcfs
```
Edit common configuration:     

```
$ sudo vim /usr/share/lxc/config/common.conf
#lxc.cgroup.devices.deny = a
lxc.cgroup.devices.allow = a
......
### /dev/random
lxc.cgroup.devices.allow = c 1:8 rwm
### tty0, tty1, tty7, tty8
lxc.cgroup.devices.allow = c 4:0 rwm
lxc.cgroup.devices.allow = c 4:1 rwm
lxc.cgroup.devices.allow = c 4:7 rwm
lxc.cgroup.devices.allow = c 4:8 rwm
......
lxc.cgroup2.devices.allow = c *:* m
lxc.cgroup2.devices.allow = b *:* m
......
### fuse
lxc.cgroup2.devices.allow = c 10:229 rwm
### customization
## graphics. /dev/dri
lxc.cgroup2.devices.allow = c 226:0 rwm
lxc.cgroup2.devices.allow = c 226:128 rwm
## graphics. /dev/fb0
lxc.cgroup2.devices.allow = c 29:0 rwm
## tty0, 1, 7, 8
lxc.cgroup2.devices.allow = c 4:0 rwm
lxc.cgroup2.devices.allow = c 4:1 rwm
lxc.cgroup2.devices.allow = c 4:7 rwm
lxc.cgroup2.devices.allow = c 4:8 rwm
......
# Setup the default mounts
#lxc.mount.auto = cgroup:mixed proc:mixed sys:mixed
lxc.mount.auto = cgroup:mixed proc:rw sys:mixed
```
Prepare the environment:    

```
chmod 777 /dev/tty* && chmod 777 -R /dev/dri/ && chmod 777 /dev/fb0
```
Create the uos lxc instance:    

```
lxc-create -t local -n uoslxc -- -m /root/meta.tar.xz -f /root/uoslxc.tar.xz
```
Edit the lxc config:    

```
# vim /var/lib/lxc/uoslxc/config

......
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
Edit the lxc guest configuration:    

```
root@zkfdhost:~# vim /var/lib/lxc/uoslxc/rootfs/etc/fstab 
# /dev/vda1 LABEL=EFI
#UUID=5474-499A      	/boot/efi 	vfat      	rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,utf8,errors=remount-ro	0 2

root@zkfdhost:~# vim /var/lib/lxc/uoslxc/rootfs/etc/lightdm/lightdm.conf 
...
[LightDM]
.....
minimum-vt=8
...
```
Start the instance:    

```
# lxc-start -n uoslxc
```
X crash because xorg is not compatible with xorg:    

```
$ cat /var/log/lightdm/x-0.log
...
(==) Log file: "/var/log/Xorg.0.log", Time: Thu Nov 14 09:35:42 2024
(==) Using config directory: "/etc/X11/xorg.conf.d"
(==) Using system config directory "/usr/share/X11/xorg.conf.d"
(EE) 
(EE) Backtrace:
(EE) 0: /usr/lib/xorg/Xorg (OsLookupColor+0x1a8) [0x599130]
(EE) 
(EE) Segmentation fault at address 0x0
(EE) 
Fatal server error:
(EE) Caught signal 11 (Segmentation fault). Server aborting
(EE) 
(EE) 
...
```
Solution, changes to Ramfb:    

![/images/20241114_093758_x.jpg](/images/20241114_093758_x.jpg)

Successful screenshot:    

![/images/20241114_093905_x.jpg](/images/20241114_093905_x.jpg)

### kylin lxc
Create via:    

```
root@zkfdhost:~# lxc-create -t local -n kylinlxc  -- -m /root/meta.tar.xz -f /root/kylinv10arm.tar.xz

```
problem:    

```
root@zkfdhost:~# lxc-start -n kylinlxc -F
[!!!!!!] Kylin kernel check failed!, freezing.
Freezing execution.

```
因为kylin是基于ubuntu16.04来做的，可以尝试将systemd替换，直接替换.    
