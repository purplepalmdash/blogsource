+++
title = "BrigedNetworkIssue"
date = "2018-08-23T21:22:56+08:00"
description = "BridgedNetworkIssue"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Problem
br0->eth0, kvm bridged to br0.    

```
br0: 192.192.189.128
kvm vm address: 192.192.189.109
vm->ping->br0, OK
vm->ping->192.192.189.24/0, Failed
```
### Investigation
Examine the forward and ebtables:   

```
# cat /proc/sys/net/ipv4/ip_forward
1
# ebtables -L
Should be ACCEPT
```
Use following command for examine the dropped package:    

```
# iptables -x -v --line-numbers -L FORWARD 

DOCKER-ISOLATION
```
Not because of the docker forward, but we have to add br0->br0 rules

### Solution
Add one rule:    

```
# iptables -A FORWARD -i br0 -o br0 -j ACCEPT
# apt-get install iptables-persistent
# vim /etc/iptables/rules.v4
*filter
-A FORWARD -i br0 -o br0 -j ACCEPT
COMMIT
```
### Further(Multicast)
Add rc.local systemd item:    

```
# vim /etc/systemd/system/rc-local.service
[Unit]
Description=/etc/rc.local
ConditionPathExists=/etc/rc.local

[Service]
Type=forking
ExecStart=/etc/rc.local start
TimeoutSec=0
StandardOutput=tty
RemainAfterExit=yes
SysVStartPriority=99

[Install]
WantedBy=multi-user.target
```
The `/etc/rc.local` should be like following:    

```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
 
exit 0
```
Use `chmod 777 /etc/rc.local` to let it executable.    

Systemd enable and run:    

```
# systemctl enable rc-local
# systemctl start rc-local
```
Enable multicast, add one line into `/etc/rc.local`:    

```
# vim /etc/rc.local
...
echo "0">/sys/class/net/br0/bridge/multicast_snooping

exit 0

```
