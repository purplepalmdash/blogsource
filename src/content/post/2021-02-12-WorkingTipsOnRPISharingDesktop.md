+++
title= "WorkingTipsOnRPISharingDesktop"
date = "2021-02-12T10:23:25+08:00"
description = "WorkingTipsOnRPISharingDesktop"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 无线接入点配置

#### 1. 基本配置
树莓派4B 8G版本，刷入了Ubuntu 20.04.2 arm64版本：    

```
ubuntu@rpi1:~$ cat /etc/issue
Ubuntu 20.04.2 LTS \n \l

ubuntu@rpi1:~$ uname -a
Linux rpi1 5.4.0-1028-raspi #31-Ubuntu SMP PREEMPT Wed Jan 20 11:30:45 UTC 2021 aarch64 aarch64 aarch64 GNU/Linux
```
#### 2. 网络配置
默认Ubuntu20.04采用netplan作为网络配置方式，一般情况下满足网络配置需求，然而在配置无线接入点的时候，需要固定wlan0 IP地址的情况下，netplan配置就不能成功，因为它在配置wlan0 固定IP地址时需要配置ssid。因而我们采用传统的ifupdown作为网络配置手段:    

关闭netplan配置:   

```
# mv /etc/netplan/50-cloud-init.yaml /root
```
安装必要的软件:    

```
# apt-get install -y resolvconf netctl ifupdown hostapd dnsmasq
```
配置网络:    

```
# vim /etc/network/interfaces
    # Include files from /etc/network/interfaces.d:
    source-directory /etc/network/interfaces.d
    
    source /etc/network/interfaces.d/*
    
    auto lo
    iface lo inet loopback
    
    
    auto eth0
    iface eth0 inet dhcp
    
    auto wlan0
    iface wlan0 inet static  
        address 10.0.70.1
            netmask 255.255.255.0
# chmod 777 /etc/network/interfaces
```
现在重新启动树莓派，发现eth0配置成功，然而wlan0尚未配置， 我们通过在crontab中配置定时任务的方法来配置wlan0:    

```
# crontab -e 
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

@reboot sleep 120 && /usr/bin/systemctl restart networking 
```
使能crontab:    

```
# systemctl enable cron
```
#### 3. hostapd配置
配置hostapd(`/etc/hostapd/hostapd.conf`):    

```
# the interface used by the AP
interface=wlan0
driver=nl80211
# "g" simply means 2.4GHz band
hw_mode=g
# the channel to use
channel=10
# limit the frequencies used to those allowed in the country
ieee80211d=1
# the country code
country_code=CN
# 802.11n support
ieee80211n=1
# QoS support
wmm_enabled=1
# the name of the AP
ssid=rpiwifi
macaddr_acl=0
# 1=wpa, 2=wep, 3=both
auth_algs=1
ignore_broadcast_ssid=0
# WPA2 only
wpa=2
wpa_passphrase=xxxxxxxxxxxxx
wpa_key_mgmt=WPA-PSK
#wpa_pairwise=TKIP
rsn_pairwise=CCMP
```
编辑文件`/etc/default/hostapd`， 更改含有`DAEMON_CONF`的行为: `DAEMON_CONF="/etc/hostapd/hostapd.conf"`.   

然而此时hostapd在启动以后并不会重新启动，我们需要在crontab中添加其自动启动.     

```
# crontab -e
@reboot sleep 120 && /usr/bin/systemctl restart networking  && systemctl restart hostapd

``` 
#### 4. IP地址配置
此时hostapd无法给客户端配置IP地址，为此我们需要配置dnsmasq(`/etc/dnsmasq.conf`):    

```
#配置监听地址
listen-address=127.0.0.1,10.0.70.1
#配置DHCP分配段
dhcp-range=10.0.70.50,10.0.70.150,12h
dhcp-option=3,10.0.70.1
```

#### 5. iptables配置
编辑/etc/sysctl.conf并取消这一行的注释：

```
net.ipv4.ip_forward=1
```
为eth0出站流量添加伪装：

```
# sudo iptables -t nat -A  POSTROUTING -o eth0 -j MASQUERADE
```
我们调节crontab为:     

```
@reboot sleep 30 && /usr/bin/systemctl restart networking  && systemctl restart hostapd && /usr/sbin/iptables -t nat -A  POSTROUTING -o eth0 -j MASQUERADE

```

到现在为止，我们应该可以配置出了一个随时可以访问internet的rpi接入点。

### 后续需要注意点
以该rpi为接入点，接入到某个网络中，然而该网络中的Internet是通过另台rpi的WIFI所共享的。   

另台RPI上的无线连接通过`wifi-menu`来配置:    

```
# apt-get install -y netctl
```
