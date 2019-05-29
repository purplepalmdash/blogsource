+++
title = "LinuxUSBEthernetBonding"
date = "2019-05-29T14:30:26+08:00"
description = "LinuxUSBEthernetBonding"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Reason
Previously, 100M->1000M,   
After upgrading: 100M+100M -> 1000M
![/images/2019_05_29_14_40_06_638x528.jpg](/images/2019_05_29_14_40_06_638x528.jpg)


### Hardware
TL-SG108E Version 1.0:    

![/images/2019_05_29_14_31_23_600x357.jpg](/images/2019_05_29_14_31_23_600x357.jpg)

Install Unmanaged pro, and use it for accesing TL-SG108E, we need to configure
LAG on switch(LAG1, 1/2, LAG2, 5,6):    

![/images/2019_05_29_14_49_01_863x529.jpg](/images/2019_05_29_14_49_01_863x529.jpg)

Laptop1 network linking:    

![/images/2019_05_29_14_52_55_860x636.jpg](/images/2019_05_29_14_52_55_860x636.jpg)

Powersync(100M/s) + D-Link(100M/s), all attaches to an usb hub, then
connecting to the laptop.    

### USB Ethernet Rename
Following configuration should be written:    

```
# pwd
/etc/systemd/network

# cat 10-ethusb1.link 
[Match]
MACAddress=xxxxxxxxxxxxxx

[Link]
Description=USB to Ethernet Adapter
Name=ethusb1
# cat 10-ethusb1.network 
[Match]
Name=ethusb1

[Network]
Address=192.168.0.33
# cat 30-ethusb2.link 
[Match]
MACAddress=8xxxxxxxxxxxxxx

[Link]
Description=USB to Ethernet Adapter 2
Name=ethusb2
# cat 30-ethusb2.network 
[Match]
Name=ethusb2

[Network]
Address=xxxxxxxxx
```
Reboot to view the configuration and examine its result via `ifconfig ethusb1`
and `ifconfig ethusb2`.     

### Bonding
Configure bond0:    

```
# pwd
/etc/systemd/network
# cat bond1.network 
[Match]
Name=bond1

[Network]
BindCarrier=ethusb1 ethusb2
# cat bond1.netdev 
[NetDev]
Name=bond1
Kind=bond

[Bond]
Mode=balance-rr
# cat Management.network 
[Match]
Name=bond1

[Network]
Address=192.168.0.33/24
```
Now you could see bond has been configured, and the transfer speed could up to
20M/s
