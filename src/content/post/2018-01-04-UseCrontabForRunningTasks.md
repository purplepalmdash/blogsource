+++
title = "UseCrontabForRunningTasks"
date = "2018-01-04T14:59:00+08:00"
description = "UseCrontabForRunningTasks"
keywords = ["Linux"]
categories = ["Technology"]
+++
If I'd wanna to run the same tasks on several machine at the certain time, I
could do following tricks:    

```
# crontab -l
# crontab /root/mycron
```
While you could set non-password login among those machines, and transfer the
same crontab files to these machine.    

```
scp /root/mycron root@192.168.10.22:/root
scp /root/mycron root@192.168.10.2X:/root
scp /root/mycron root@192.168.10.2X:/root
scp /root/mycron root@192.168.10.2X:/root
scp /root/mycron root@192.168.10.2X:/root
```
You cron file would be seen like following:    

```
# cat /root/mycron
59 14 * * * /root/benchmark.sh
```
this means the scripts of `/root/benchmark.sh` would be run at 14:59 AM. In
this file you could do whatever you want.    

![/images/2018_01_04_17_18_39_448x169.jpg](/images/2018_01_04_17_18_39_448x169.jpg)

### SomeTips For Virtualbox
[https://www.virtualbox.org/manual/ch09.html](https://www.virtualbox.org/manual/ch09.html)    

```
9.11.4. Binding NAT sockets to a specific interface
By default, VirtualBox's NAT engine will route TCP/IP packets through the default interface assigned by the host's TCP/IP stack. (The technical reason for this is that the NAT engine uses sockets for communication.) If, for some reason, you want to change this behavior, you can tell the NAT engine to bind to a particular IP address instead. Use the following command:

VBoxManage modifyvm "VM name" --natbindip1 "10.45.0.2"
After this, all outgoing traffic will be sent through the interface with the IP address 10.45.0.2. Please make sure that this interface is up and running prior to this assignment.
```

Some reference topics:    

[https://forums.virtualbox.org/viewtopic.php?f=6&t=81631](https://forums.virtualbox.org/viewtopic.php?f=6&t=81631)    

[https://forums.virtualbox.org/viewtopic.php?f=1&t=38879](https://forums.virtualbox.org/viewtopic.php?f=1&t=38879)    
