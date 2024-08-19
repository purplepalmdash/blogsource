+++
title= "PVEOnHygon"
date = "2024-08-19T08:23:51+08:00"
description = "PVEOnHygon"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Host configuration
Hardware Info:    

```
# cat /proc/cpuinfo | grep -i "model name"
model name	: Hygon C86 3350  8-core Processor
```
Os Info:    

```
root@pve:~# cat /etc/debian_version 
12.4
root@pve:~# uname -a
Linux pve 6.5.11-8-pve #1 SMP PREEMPT_DYNAMIC PMX 6.5.11-8 (2024-01-30T12:27Z) x86_64 GNU/Linux
```
Modifications :      

```
# cat /etc/default/grub | grep amd_iommu
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt amd_iommu=on initcall_blacklist=sysfb_init pcie_acs_override=downstream,multifunction"
# cat /etc/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
# cat /etc/modprobe.d/vfio.conf
options vfio-pci ids=1002:6611,1002:aab0
options vfio-pci disable_idle_d3=1
# update-initramfs -u -k all && update-grub && reboot
```
### virtualization
Win10 configuration:    

![/images/20240819_082807_x.jpg](/images/20240819_082807_x.jpg)

you can get win10 working properly directly, but for win7, it could only boot into system, after driver installation, it will remains black and could not startup.    
