+++
title= "PreProductionCentOS76Libvirtd"
date = "2023-06-19T17:21:11+08:00"
description = "PreProductionCentOS76Libvirtd"
keywords = ["Technology"]
categories = ["Technology"]
+++
### System Information
Basic info:   

```
[root@tl ~]# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
[root@tl ~]# uname -r
3.10.0-957.el7.x86_64
[root@tl ~]# lscpu |grep "Model name"
Model name:            12th Gen Intel(R) Core(TM) i3-12100
[root@tl ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:          64005         482       63299           9         223       63023
Swap:          2043           0        2043
```
### Kernel
Install kernel and configure its default startup:    

```
[root@tl materials]# cp -ar i915/* /lib/firmware/i915/
cp: overwrite ‘/lib/firmware/i915/bxt_dmc_ver1_07.bin’? y
cp: overwrite ‘/lib/firmware/i915/bxt_dmc_ver1.bin’? y
cp: overwrite ‘/lib/firmware/i915/bxt_guc_ver8_7.bin’? y
......
# rpm -ivh kernel-*.rpm
```
