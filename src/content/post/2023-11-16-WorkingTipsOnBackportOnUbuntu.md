+++
title= "WorkingTipsOnBackportOnUbuntu"
date = "2023-11-16T09:24:52+08:00"
description = "WorkingTipsOnBackportOnUbuntu"
keywords = ["Technology"]
categories = ["Technology"]
+++
Steps:    

```
git clone https://github.com/intel-gpu/intel-gpu-i915-backports.git
cd intel-gpu-i915-backports/
git checkout backport/main
```
Switch to hwe kernel:    

```
$ sudo apt install linux-generic-hwe-22.04
$ uname -r
6.2.0-36-generic
```
Install packages:     

```
sudo apt install dkms make debhelper devscripts build-essential flex bison mawk

```
