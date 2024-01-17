+++
title= "WorkingTipsOnRongHe"
date = "2024-01-15T09:02:46+08:00"
description = "WorkingTipsOnRongHe"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Tips
crontab items:    

```
@reboot /usr/bin/execpipe.sh
```
`execpipe` content:     

```
$ cat /usr/bin/execpipe.sh 
#!/bin/bash
while true; do eval "$(cat  /mypipe)" &> /mypipeoutput.txt;done
#while true; do eval "$(cat  /mypipe)";done
```
Create the pipe via:    

```
$ ls / | grep mypipe
mypipe
mypipeoutput.txt
```

### Kernel Building(VB)
Build the kernel via:    

```
apt install -y git fakeroot build-essential ncurses-dev xz-utils libssl-dev bc flex libelf-dev bison rsync kmod cpio unzip
unzip kernel-config.zip
cp kernel-config/x86_64_defconfig .config
./scripts/config --disable DEBUG_INFO
echo "" | make ARCH=x86_64 olddefconfig
make ARCH=x86_64 -j16 LOCALVERSION=-lts2021-iotg  bindeb-pkg
```
Kernel patch backport:     

```
drivers/gpu/drm/i915/display/intel_fbc.c, line 1029, not equal to tc's implementation

/drivers/gpu/drm/i915# vim i915_driver.c, 存在较大不同
```
