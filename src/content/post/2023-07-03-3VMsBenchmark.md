+++
title= "3VMsBenchmark"
date = "2023-07-03T17:13:03+08:00"
description = "3VMsBenchmark"
keywords = ["Technology"]
categories = ["Technology"]
+++
Create 3 disk files:    

```
 qemu-img create -f qcow2 -b win.qcow2 -F qcow2 vm1.qcow2
 qemu-img create -f qcow2 -b win.qcow2 -F qcow2 vm2.qcow2
 qemu-img create -f qcow2 -b win.qcow2 -F qcow2 vm3.qcow2
```
Create vm1/vm2/vm3.   

Start and detect the spice:    

```
netstat -anp | grep 590
tcp        0      0 0.0.0.0:5902            0.0.0.0:*               LISTEN      7307/qemu-system-x8 
tcp        0      0 0.0.0.0:5900            0.0.0.0:*               LISTEN      6813/qemu-system-x8 
tcp        0      0 0.0.0.0:5901            0.0.0.0:*               LISTEN      7101/qemu-system-x8 

```
