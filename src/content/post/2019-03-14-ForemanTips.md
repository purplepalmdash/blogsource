+++
title = "ForemanTips"
date = "2019-03-14T15:18:03+08:00"
description = "ForemanTips"
keywords = ["Linux"]
categories = ["Linux"]
+++
### System
Install Ubuntu 18.04, 4 Core/ 4G memory, 50 G disk.    

Install with basic sshd support.   

![/images/2019_03_14_15_20_12_676x259.jpg](/images/2019_03_14_15_20_12_676x259.jpg)

Network planning:    
10.192.189.0/24, no dhcp.    

### Network
Configure the networking via following commands:    

```
# vim /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: no
      addresses: [10.192.189.2/24]
      gateway4: 10.192.189.1
      nameservers:
        addresses: [223.5.5.5,180.76.76.76]

# netplan --debug apply
# systemctl disable systemd-resolved.service
# systemctl stop systemd-resolved.service
# rm -f /etc/resolv.conf 
# echo nameserver 223.5.5.5>/etc/resolv.conf
```
Configure the hostname:    

```
# sudo hostnamectl set-hostname foreman.fuck.com
# echo "10.192.189.2 foreman.fuck.com" | sudo tee -a /etc/hosts
```
### Install foreman
Install foreman via following commands:    

```
......
```
Refers to:    

![https://computingforgeeks.com/how-to-install-foreman-on-ubuntu-18-04-lts-bionic-beaver/](https://computingforgeeks.com/how-to-install-foreman-on-ubuntu-18-04-lts-bionic-beaver/)    


