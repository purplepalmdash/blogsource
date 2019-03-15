+++
title = "UsbNetworkCard"
date = "2019-03-14T09:54:29+08:00"
description = "UsbNetworkCard"
keywords = ["Linux"]
categories = ["Linux"]
+++
Using systemd-networkd for configurating the usb network card,     

```
# vim /etc/systemd/nework/10-ethusb1.link
[Match]
MACAddress=00:xx:xx:.....

[Link]
Description=USB to Ethernet Adapter
Name=ethusb1
```
Then configurating the ethusb1 ip address:    

```
# vim /etc/systemd/network/10-ethusb1.network 
[Match]
Name=ethusb1

[Network]
Address=192.168.0.33
```
Reboot the computer then you could see the ethusb1 available.   
