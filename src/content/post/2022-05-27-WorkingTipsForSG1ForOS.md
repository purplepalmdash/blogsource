+++
title= "WorkingTipsForSG1ForOS"
date = "2022-05-27T08:42:18+08:00"
description = "WorkingTipsForSG1ForOS"
keywords = ["Technology"]
categories = ["Technology"]
+++
Create disk via:    

```
 qemu-img create -f qcow2 -b /images/Centos1810Base.qcow2 sg1_openstack.qcow2
```
Create a vm, using bridged networking and set its ip address to `192.168.89.25`.   
Upload the kernel and deployment files onto the server:    

```
scp ./kernel_4.19.12-1.xxx_rpms.tar.gz SG1Deployment.tar.gz ctctest@192.168.89.25:~
``` 
To 25, and install the kernel via:     

```
# ./install_kernel.sh
# reboot
```
After reboot, check the kernel version.   
