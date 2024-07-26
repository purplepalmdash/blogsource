+++
title= "JumperConfiguration"
date = "2024-07-26T14:03:45+08:00"
description = "JumperConfiguration"
keywords = ["Technology"]
categories = ["Technology"]
+++
Ubuntu22.04 Install desktop version.     

```
sudo apt update -y
sudo apt upgrade -y
sudo apt install -y openssh-server tigervnc-standalone-server tigervnc-xorg-extension lxqt vim net-tools curl
sudo systemctl set-default multi-user.target
sudo reboot
```
Configure vnc:     

```
$ vncpasswd
```

Configure the vnc:      

```
test@jumper:~$ cat ~/.vnc/config 
session=lxqt
geometry=1920x1080
localhost=no
alwaysshared
test@jumper:~$ cat /etc/tigervnc/vncserver.users 
# TigerVNC User assignment
#
# This file assigns users to specific VNC display numbers.
# The syntax is <display>=<username>. E.g.:
#
# :2=andrew
# :3=lisa
:1=test
test@jumper:~$ sudo systemctl enable tigervncserver@:1
Created symlink /etc/systemd/system/multi-user.target.wants/tigervncserver@:1.service → /lib/systemd/system/tigervncserver@.service.
```
download the citrix workspace from websiste, and install them via:      

```
sudo dpkg -i icaclient_24.5.0.76_amd64.deb 
sudo dpkg -i ctxusb_24.5.0.76_amd64.deb 
```
Configure network in network manager.    

```
ens160: static ip 192.168.1.33
ens192: dhcp from company networking. 
```
Add crontab for sharing:      

```
root@jumper:/home/test# crontab -l
@reboot sleep 10 && /usr/bin/startsharing.sh
root@jumper:/home/test# cat /usr/bin/startsharing.sh
#!/bin/sh -e
iptables -t nat -A POSTROUTING -o ens192 -j MASQUERADE
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i ens160 -o ens192 -j ACCEPT
echo "1" > /proc/sys/net/ipv4/ip_forward
```
