+++
title = "UseCrontabForRunningTasks"
date = "2018-01-04T14:59:00+08:00"
description = "UseCrontabForRunningTasks"
keywords = ["Linux"]
categories = ["Linux"]
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
