+++
title = "InternetSharingForRPI"
date = "2018-09-04T16:40:34+08:00"
description = "InternetSharingForRPI"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Diagram

![/images/2018_09_04_16_41_05_678x430.jpg](/images/2018_09_04_16_41_05_678x430.jpg)

### Command
RPI: 192.168.0.16/24    
Laptop USB Ethernet Adapter: 192.168.0.33/24    


Laptop:    

```
# sudo iptables -t nat -A POSTROUTING -s 192.168.0.16/24 ! -d 192.168.0.16/24 -j MASQUERADE
```
Rpi:    

```
# sudo route delete default gw 192.168.0.1
# sudo route add default gw 192.168.0.33
# sudo vim /etc/resolv.conf
nameserver 192.168.42.129
```
Thus your rpi could directly go to the internet.    
