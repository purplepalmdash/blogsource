---
categories: ["virtualization"]
comments: true
date: 2015-07-22T09:54:59Z
title: Libvirtd Trouble-Shooting On CentOS7.1
url: /2015/07/22/libvirtd-trouble-shooting-on-centos7-dot-1/
---

### Problem Description
When upgraded from CentOS6.6 to CentOS7, the libvirtd will encounter following error:    

![/images/2015_07_22_09_54_37_355x184.jpg](/images/2015_07_22_09_54_37_355x184.jpg)     

Simply remove: 

```
# virsh edit nodename
-     <feature policy='require' name='invtsc'/>
```
