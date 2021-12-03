+++
title= "WorkingTipsOnSG1OnUbuntu21.10"
date = "2021-11-30T14:39:28+08:00"
description = "WorkingTipsOnSG1OnUbuntu21.10"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Info
Displayed system info via:    

```
test@sg1:~$ sudo inxi
CPU: 16x Single Core Intel Xeon (Cascadelake) (-SMP-) speed: 2993 MHz Kernel: 5.13.0-19-generic x86_64 Up: 4m 
Mem: 1352.3/32102.2 MiB (4.2%) Storage: 100 GiB (8.8% used) Procs: 344 Shell: Bash inxi: 3.3.06 
test@sg1:~$ sudo inxi -G
Graphics:  Device-1: Red Hat QXL paravirtual graphic card driver: qxl v: kernel 
           Device-2: Intel SG1 [Server GPU SG-18M] driver: N/A 
           Display: server: X.org 1.20.13 driver: loaded: N/A tty: 190x40 
           Message: Advanced graphics data unavailable in console for root. 
test@sg1:~$ cat /etc/issue
Ubuntu 21.10 \n \l
```

