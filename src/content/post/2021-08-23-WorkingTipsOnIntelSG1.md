+++
title= "WorkingTipsOnIntelSG1"
date = "2021-08-23T11:11:48+08:00"
description = "WorkingTipsOnIntelSG1"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Hardware & OS
Refers to:    

`https://www.h3c.com/en/Support/Resource_Center/EN/Severs/Catalog/Optional_Parts/GPU/Technical_Documents/Install/User_Guide/H3C_XG310_GPU_UG-5W100/`

```
# cat /etc/issue
Ubuntu 20.04.1 LTS \n \l

# uname -a
Linux xxxxxxxxxxxxxxxxxx 5.4.48+ #1 SMP Wed Feb 3 10:57:04 CST 2021 x86_64 x86_64 x86_64 GNU/Linux
```
CPU and memory:    

```
# cat /proc/cpuinfo
...
processor       : 95
vendor_id       : GenuineIntel
cpu family      : 6
model           : 85
 model name      : Intel(R) Xeon(R) Gold 6248R CPU @ 3.00GHz
...
# free -g
              total        used        free      shared  buff/cache   available
Mem:            503           3         496           0           3         497
Swap:             1           0           1
```
### Steps
Steps:    

```
# mkdir IntelAndroid
# tar xzvf xxxxxxxxxxxx.tar.gz

```
