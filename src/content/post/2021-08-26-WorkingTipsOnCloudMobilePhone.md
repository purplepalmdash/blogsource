+++
title= "WorkingTipsOnCloudMobilePhone"
date = "2021-08-26T11:32:10+08:00"
description = "WorkingTipsOnCloudMobilePhone"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 步骤
Install CentOS 7.4 :    

![/images/2021_08_26_11_32_24_795x590.jpg](/images/2021_08_26_11_32_24_795x590.jpg)

With route:    

```
[root@intelandroid ctctest]# ip route
default via 192.168.91.254 dev enp61s0f0 proto static metric 100 
192.168.89.0/24 dev enp61s0f0 proto kernel scope link src 192.168.89.108 metric 100 
192.168.91.254 dev enp61s0f0 proto static scope link metric 100 
[root@intelandroid ctctest]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp61s0f0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq portid 3cd2e55e05da state UP qlen 1000
    link/ether 3c:d2:e5:5e:05:da brd ff:ff:ff:ff:ff:ff
    inet 192.168.89.108/24 brd 192.168.89.255 scope global enp61s0f0
       valid_lft forever preferred_lft forever
    inet6 fe80::1e1f:7e4d:5f9f:a2ab/64 scope link 
       valid_lft forever preferred_lft forever
```
Update grub:    

```
# vim /etc/default/grub
...
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rhgb quiet modprobe.blacklist=ast"
...
# grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
# reboot
```
After reboot, check ast module is not loaded:     

```
# lsmod | grep ast
Should be nothing here. 
```

### 
