+++
title= "WorkingTipsOnRPIvpnServer"
date = "2023-02-12T08:47:04+08:00"
description = "WorkingTipsOnRPIvpnServer"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 变更(piap->pivpn)
主要修改如下:     
本来用来做公司的AP,但是因为需要，弄成了家里的pivpn, 
1. 关闭了hostapd: 
sudo systemctl disable hostapd
2. 关闭了xray
sudo systemctl disable xray
3. 关闭了shadowsocks
sudo systemctl disable shadowsocks
4. 关闭了pdnsd
sudo systemctl disable pdnsd
5. 关闭了dhcpd
sudo systemctl disable isc-dhcp-server
6. 关闭了motion(摄像头)
sudo systemctl disable motion 
7. 关闭了shadowsocks-libev
sudo systemctl disable shadowsocks-libev

更改了网络配置:    

```
$ cat /etc/network/interfaces
# interfaces(5) file used by ifup(8) and ifdown(8)

# Please note that this file is written to be used with dhcpcd
# For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'

# Include files from /etc/network/interfaces.d:
#source-directory /etc/network/interfaces.d
#auto wlan0
#iface wlan0 inet static
#  address 10.16.42.1
#  netmask 255.255.255.0
auto eth0
iface eth0 inet static
  address 192.168.1.117
  netmask 255.255.255.0
  gateway 192.168.1.1
```

更新到最新(2023.02.12)

