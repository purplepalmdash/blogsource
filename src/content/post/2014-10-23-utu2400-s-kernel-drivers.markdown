---
categories: ["embedded"]
comments: true
date: 2014-10-23T00:00:00Z
title: utu2400's Kernel Drivers
url: /2014/10/23/utu2400-s-kernel-drivers/
---

### sshd replacement
Since busybox enabled the telnetd by default, we could just use telnet for accessing the board.    

```
[root@www ~]# ps -ef | grep telnet
  861 root       0:00 /usr/sbin/telnetd -l /bin/login
  893 root       0:00 grep telnet

```
### 

