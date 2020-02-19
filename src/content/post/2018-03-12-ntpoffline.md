+++
title = "ntpoffline"
date = "2018-03-12T22:01:37+08:00"
description = "ntpoffline"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Server
server side configuration:    

```
# yum install -y ntpd
# vim /etc/ntpd.conf
```
The configuration file is listed as following:   

```
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1 
restrict ::1
restrict 192.168.0.0 mask 255.255.0.0 nomodify notrap
server 127.127.1.0  # local clock
fudge 127.127.1.0  stratum 10
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```
Disable the chronyd service thus the ntpd could acts properly:   

```
# systemctl disable chronyd
# systemctl enable ntpd
# systemctl start ntpd
# systemctl disable firewalld
```
### Client
Install via:    

```
# yum install -y ntpd
```
Configuration file:    

```
driftfile /var/lib/ntp/drift
restrict default nomodify notrap nopeer noquery
restrict 127.0.0.1 
restrict ::1
server 192.168.122.200
# 配置允许上游时间服务器主动修改本机的时间
restrict 192.168.122.200 nomodify notrap noquery
includefile /etc/ntp/crypto/pw
keys /etc/ntp/keys
disable monitor
```
Also disable the chronyd service and enable the ntpd service. The client will
automatically sync with the server `192.168.122.200`.     
