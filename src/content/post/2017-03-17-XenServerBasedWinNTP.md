+++
date = "2017-03-17T16:58:26+08:00"
title = "XenServer Windows NTP时间配置"
categories = ["Linux"]
keywords = ["Linux"]
description = "sync time using ntp in XenServer Windows"

+++
### ntp server
Configure an ntp server in ubuntu16.04, IP address is `192.168.0.221`:    

```
$ sudo apt-get install -y openntp
$ sudo vim /etc/ntp.conf
```
The reference configuration file is listed as following:    

```
# /etc/ntp.conf, configuration for ntpd; see ntp.conf(5) for help

driftfile /var/lib/ntp/ntp.drift

# Enable this if you want statistics to be logged.
#statsdir /var/log/ntpstats/

statistics loopstats peerstats clockstats
filegen loopstats file loopstats type day enable
filegen peerstats file peerstats type day enable
filegen clockstats file clockstats type day enable

# Specify one or more NTP servers.

# Use servers from the NTP Pool Project. Approved by Ubuntu Technical Board
# on 2011-02-08 (LP: #104525). See http://www.pool.ntp.org/join.html for
# more information.
#pool 0.ubuntu.pool.ntp.org iburst
#pool 1.ubuntu.pool.ntp.org iburst
#pool 2.ubuntu.pool.ntp.org iburst
#pool 3.ubuntu.pool.ntp.org iburst

server 0.cn.pool.ntp.org
server 1.cn.pool.ntp.org
server 2.cn.pool.ntp.org
server 3.cn.pool.ntp.org
#
#server 127.127.1.0 minpoll 4
#fudge 127.127.1.0 stratum 1
#restrict 127.0.0.1

# Use Ubuntu's ntp server as a fallback.
#pool ntp.ubuntu.com

# Access control configuration; see /usr/share/doc/ntp-doc/html/accopt.html for
# details.  The web page <http://support.ntp.org/bin/view/Support/AccessRestrictions>
# might also be helpful.
#
# Note that "restrict" applies to both servers and clients, so a configuration
# that might be intended to block requests from certain clients could also end
# up blocking replies from your own upstream servers.

# By default, exchange time with everybody, but don't allow configuration.
restrict -4 default kod notrap nomodify nopeer noquery limited
restrict -6 default kod notrap nomodify nopeer noquery limited

# Local users may interrogate the ntp server more closely.
restrict 127.0.0.1
restrict ::1

# Needed for adding pool entries
restrict source notrap nomodify noquery

# Clients from this (example!) subnet have unlimited access, but only if
# cryptographically authenticated.
#restrict 192.168.123.0 mask 255.255.255.0 notrust
restrict 192.168.1.0 mask 0.0.0.0 nomodify notrap
```
Then restart the ntp service via:    

```
$ sudo systemctl restart ntp
```
Note the ubuntu16.04 machine should be online(could reach Internet), after a while it will sync the 
time from the internet.    

### Windows Configuration
See following image:    

![/images/2017_03_17_17_01_39_515x518.jpg](/images/2017_03_17_17_01_39_515x518.jpg)


