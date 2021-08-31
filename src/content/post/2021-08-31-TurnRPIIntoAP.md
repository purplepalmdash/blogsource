+++
title= "TurnRPIIntoAP"
date = "2021-08-31T09:35:09+08:00"
description = "TurnRPIIntoAP"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 目的
将闲置的RPI变为一个AP,有线转无线，用于快速连接网络开发。
### 准备材料
下载`2021-05-07-raspios-buster-armhf-lite.zip`， 解压并写入SD卡，之后用SD卡启动RPI3, 默认用户名及密码是`pi/raspberry`, 写入后使用`rpi-config`扩充文件系统：    

```
pi@raspberrypi:~ $ uname -a
Linux raspberrypi 5.10.52-v7+ #1441 SMP Tue Aug 3 18:10:09 BST 2021 armv7l GNU/Linux
pi@raspberrypi:~ $ cat /etc/issue
Raspbian GNU/Linux 10 \n \l
```
### 步骤
更改为tsinghua的源以加速:    

```
# 编辑 `/etc/apt/sources.list` 文件，删除原文件所有内容，用以下内容取代：
deb http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib rpi
deb-src http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ buster main non-free contrib rpi

# 编辑 `/etc/apt/sources.list.d/raspi.list` 文件，删除原文件所有内容，用以下内容取代：
deb http://mirrors.tuna.tsinghua.edu.cn/raspberrypi/ buster main ui
```
下载hostapd及dhcp服务器:    

```
# sudo apt-get update -y
# sudo apt-get upgrade -y
# sudo apt-get install hostapd isc-dhcp-server iptables-persistent
```
配置DHCP服务器:    

```
$ sudo vim /etc/dhcp/dhcpd.conf
```
找到以下行(这里需要注释掉默认的选项):   

```
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;
```
替换为:    

```
#option domain-name "example.org";
#option domain-name-servers ns1.example.org, ns2.example.org;
```
找到以下行(这里是激活authoritative选项):    

```
# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;
```
替换为:    

```
# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;
```

在文件的末尾添加以下定义:     

```
subnet 172.16.42.0 netmask 255.255.255.0 {
	range 172.16.42.10 172.16.42.50;
	option broadcast-address 172.16.42.255;
	option routers 172.16.42.1;
	default-lease-time 600;
	max-lease-time 7200;
	option domain-name "local";
	option domain-name-servers 8.8.8.8, 8.8.4.4;
}
```
现在保存后退出。   
现在编辑`isc-dhcp-server`的默认定义文件，配置其监听的端口:     

```
# sudo vim /etc/default/isc-dhcp-server
.....
INTERFACESv4="wlan0"
INTERFACESv6="wlan0"
````
编辑wlan0的静态地址(这里我们顺便设置了eth0的静态地址):     

```
# sudo vim /etc/network/interfaces
auto wlan0
iface wlan0 inet static
  address 172.16.42.1
  netmask 255.255.255.0
auto eth0
iface eth0 inet static
  address 192.168.1.117
  netmask 255.255.255.0
  gateway 192.168.1.33
```
配置hostapd:    

```
# sudo vim /etc/hostapd/hostapd.conf
country_code=US
interface=wlan0
driver=nl80211
ssid=Pi_AP
country_code=US
hw_mode=g
channel=6
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=xxxxxxxxxxxx
wpa_key_mgmt=WPA-PSK
wpa_pairwise=CCMP
wpa_group_rekey=86400
ieee80211n=1
wme_enabled=1
```
配置hostapd的默认配置文件:     

```
# sudo vim /etc/default/hostapd
Find the line #DAEMON_CONF="" and edit it so it says DAEMON_CONF="/etc/hostapd/hostapd.conf"
```
配置NAT（网络地址转换) :    

```
# sudo vim /etc/sysctl.conf

pi@raspberrypi:~ $ cat /etc/sysctl.conf  | grep ip_forward
net.ipv4.ip_forward=1
```
最后保存iptables:      

```
sudo iptables -t nat -S
sudo iptables -S
sudo sh -c "iptables-save > /etc/iptables/rules.v4"
```
iptables-persistent会在启动的时候自动载入保存的规则。

如此则可以将RPi3作为一个无线接入点来使用。
