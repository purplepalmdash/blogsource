+++
categories = ["Linux"]
date = "2016-10-24T16:36:24+08:00"
description = "Monitoring cloudstack VR via collectd"
keywords = ["Linux"]
title = "VRMonitoringViaCollectd"

+++
### Environment
The VR's information is listed as following:    

![/images/2016_10_24_16_35_58_501x547.jpg](/images/2016_10_24_16_35_58_501x547.jpg)    
Now login the VR in CloudStack Agent Node via:    

```
# ssh -i /root/.ssh/id_rsa.cloud -p3922 169.254.0.129
```
Modify its `/etc/apt/sources.list` like following:    

```
# cat /etc/apt/sources.list
# 

# deb cdrom:[Debian GNU/Linux 7.8.0 _Wheezy_ - Official amd64 NETINST Binary-1
20150110-14:41]/ wheezy main

#deb cdrom:[Debian GNU/Linux 7.8.0 _Wheezy_ - Official amd64 NETINST Binary-1
20150110-14:41]/ wheezy main


deb http://mirrors.aliyun.com/debian wheezy main
deb-src http://mirrors.aliyun.com/debian wheezy main

deb http://mirrors.aliyun.com/ wheezy/updates main
deb-src http://mirrors.aliyun.com/ wheezy/updates main

# wheezy-updates, previously known as 'volatile'
deb http://mirrors.aliyun.com/debian wheezy-updates main
deb-src http://mirrors.aliyun.com/debian wheezy-updates main
deb http://mirrors.aliyun.com/debian/ wheezy-backports main
```
Then `apt-get update && apt-get install -y collectd`, for installing collectd.    

### Collectd
Configuration of collectd:    

```
$ touch /var/log/collectd.log
$ vim /etc/collectd/collectd.conf
Hostname "WeiLan"
LoadPlugin logfile
#LoadPlugin syslog

<Plugin logfile>
	LogLevel "info"
	File "/var/log/collectd.log"
	Timestamp true
	PrintSeverity false
</Plugin>

#<Plugin syslog>
#	LogLevel info
#</Plugin>

..... 

LoadPlugin conntrack
LoadPlugin uptime 
#LoadPlugin df
LoadPlugin write_graphite

<Plugin write_graphite>
	<Carbon>
		Host "192.168.1.79"
		Port "2003"
		Prefix "collectd"
		Postfix "collectd"
		StoreRates false
		AlwaysAppendDS false
		EscapeCharacter "_"
	</Carbon>
</Plugin>

```
Start the collectd service via:    

```
# service collectd restart
```
