+++
title= "WorkingTipsOniventoy"
date = "2023-08-09T09:41:30+08:00"
description = "WorkingTipsOniventoy"
keywords = ["Technology"]
categories = ["Technology"]
+++
Create the server:     

```
cd Downloads
mkdir iventory
cd iventory
tar xzvf ../iventoy-1.0.17-linux-free.tar.gz
cd iventoy-1.0.17
sudo ./iventoy.sh start
```
Select the DHCP server mode for external:    

![/images/2023_08_09_09_42_48_888x625.jpg](/images/2023_08_09_09_42_48_888x625.jpg)

Create the linked iso:   

```
➜  ~ ls /home/dash/Downloads/iventory/iventoy-1.0.17/iso 
➜  ~ ln -s /media/121/iso/Rocky-9.2-x86_64-dvd.iso /home/dash/Downloads/iventory/iventoy-1.0.17/iso/Rocky-9.2-x86_64-dvd.iso
➜  ~ ln -s /media/121/iso/ubuntu-22.04.2-desktop-amd64.iso /home/dash/Downloads/iventory/iventoy-1.0.17/iso/ubuntu-22.04.2-desktop-amd64.iso
➜  ~ ls -l -h /home/dash/Downloads/iventory/iventoy-1.0.17/iso 
total 0
lrwxrwxrwx 1 dash root 39 Aug  9 09:40 Rocky-9.2-x86_64-dvd.iso -> /media/121/iso/Rocky-9.2-x86_64-dvd.iso
lrwxrwxrwx 1 dash root 47 Aug  9 09:40 ubuntu-22.04.2-desktop-amd64.iso -> /media/121/iso/ubuntu-22.04.2-desktop-amd64.iso
```
Start the server:    

![/images/2023_08_09_09_43_26_1079x379.jpg](/images/2023_08_09_09_43_26_1079x379.jpg)

Using external dhcp server may need other configuration.  

```
# vim /etc/dhcpd.conf
......
 subnet 192.168.1.0	netmask 255.255.255.0 {
    range 192.168.1.160 192.168.1.199;
    option routers 192.168.1.33;
    option subnet-mask 255.255.255.0;
       filename "iventoy_loader_16000";
       next-server 192.168.1.12;
    option broadcast-address 192.168.1.255;
    option domain-name-servers 223.5.5.5;
    option time-offset 0;
    default-lease-time	1209600;
    max-lease-time 1814400;
    }
......
``` 
