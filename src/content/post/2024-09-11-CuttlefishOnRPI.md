+++
title= "CuttlefishOnRPI"
date = "2024-09-11T09:30:53+08:00"
description = "CuttlefishOnRPI"
keywords = ["Technology"]
categories = ["Technology"]
+++
Change sshd configuration:    

```
$ sudo cat /etc/ssh/sshd_config.d/50-cloud-init.conf 
PasswordAuthentication yes
$ sudo sed -i 's@//ports.ubuntu.com@//mirrors.ustc.edu.cn@g' /etc/apt/sources.list.d/ubuntu.sources
```
`sudo apt update -y && sudo apt upgrade -y` , then reboot.   

Build cuttlefish:    

```
git clone https://github.com/google/android-cuttlefish
cd android-cuttlefish
tools/buildutils/build_packages.sh
```
