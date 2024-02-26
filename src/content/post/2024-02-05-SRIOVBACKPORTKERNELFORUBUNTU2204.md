+++
title= "SRIOVBACKPORTKERNELFORUBUNTU2204"
date = "2024-02-05T16:17:34+08:00"
description = "SRIOVBACKPORTKERNELFORUBUNTU2204"
keywords = ["Technology"]
categories = ["Technology"]
+++
Hardware/OS information:    

```
dash@alder:~$ uname -a
Linux alder 5.15.0-92-generic #102-Ubuntu SMP Wed Jan 10 09:33:48 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
dash@alder:~$ lscpu | grep -i model
Model name:                         12th Gen Intel(R) Core(TM) i3-12100
Model:                              151
```
Install prerequisite packages:    

```
sudo apt install dkms make debhelper devscripts build-essential flex bison mawk
```
Checkout the firmware:     

```
git clone https://github.com/intel-gpu/intel-gpu-firmware
cd intel-gpu-firmware
sudo mkdir -p /lib/firmware/updates/i915/
sudo cp firmware/*.bin /lib/firmware/updates/i915/
```
Checkout the src code:    

```
cd Code
git clone https://github.com/intel-gpu/intel-gpu-i915-backports/
cd intel-gpu-i915-backports/
git checkout backport/main
make i915dkmsdeb-pkg
sudo dpkg -i ../intel-i915-dkms_1.23.9.11.231003.15+i1-1_all.deb
```
Check the dkms status:    

```
$ dkms status
intel-i915-dkms/1.23.9.11.231003.15, 5.15.0-92-generic, x86_64: installed
$ sudo reboot
$ sudo dmesg |grep -i backport
[    1.336656] COMPAT BACKPORTED INIT
[    1.337218] Loading modules backported from I915-23.9.11
[    1.337702] Backport generated by backports.git I915_23.9.11_PSB_231003.15
[    1.380192] [drm] I915 BACKPORTED INIT 
[    1.421899]  __init_backport+0x47/0xfa [i915]

```
Add kernel options:    

```
GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on i915.enable_guc=3 i915.max_vfs=7"
```
Got the unstable conditions(once the dkms is installed, then cannot reboot).       