+++
title = "WorkingTipsOnPlayWithDockerUbuntu1804"
date = "2018-05-24T15:41:00+08:00"
description = "WorkingTipsOnPlayWithDockerUbuntu1804"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Basic Environment
Ubuntu 18.04, minimum installation.    

```
# vim /etc/netplan/01-netcfg.yaml 
	# This file describes the network interfaces available on your system
	# For more information, see netplan(5).
	network:
	 version: 2
	 renderer: networkd
	 ethernets:
	   eth0:
	     dhcp4: no
	     dhcp6: no
	     addresses: [192.192.189.114/24]
	     gateway4: 192.192.189.1
	     nameservers:
	       addresses: [223.5.5.5,180.76.76.76]
# sudo netplan --debug apply
# apt-get update && apt-get install -y docker.io docker-compose
```
Disable systemd-resolved:    

```
# systemctl disable systemd-resolved.service
# systemctl stop systemd-resolved.service
# echo nameserver 192.168.0.15>/etc/resolv.conf
# chattr -e /etc/resolv.conf
# chattr +i /etc/resolv.conf
```


