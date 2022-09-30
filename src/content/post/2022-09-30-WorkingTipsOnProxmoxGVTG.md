+++
title= "WorkingTipsOnProxmoxGVTG"
date = "2022-09-30T09:43:37+08:00"
description = "WorkingTipsOnProxmoxGVTG"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Change configuration
Change grub options:    

```
# vi /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
TO
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt i915.enable_gvt=1"

# vi /etc/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
kvmgt
```
Update:    

```
# update-grub2
# update-initramfs -u -k all
```
### Check status:    
Check iommu features:   

```
# dmesg | grep -e DMAR -e IOMMU -e AMD-Vi
# find /sys/kernel/iommu_groups/ -type l
```
Get the VGA related:   

```
root@pve:~# lspci | grep -i vga
00:02.0 VGA compatible controller: Intel Corporation HD Graphics 630 (rev 04)
root@pve:~# ls /sys/bus/pci/devices/0000\:00\:02.0/mdev_supported_types/
i915-GVTg_V5_4	i915-GVTg_V5_8
```
### vm installation
Create a vm (win7) like:    

![/images/2022_09_30_10_01_06_720x245.jpg](/images/2022_09_30_10_01_06_720x245.jpg)

![/images/2022_09_30_10_02_04_724x216.jpg](/images/2022_09_30_10_02_04_724x216.jpg)

Add PCI device:   

![/images/2022_09_30_10_03_54_614x173.jpg](/images/2022_09_30_10_03_54_614x173.jpg)

hardware:    

![/images/2022_09_30_10_04_22_961x349.jpg](/images/2022_09_30_10_04_22_961x349.jpg)

Then install the windows in console.    

Install "Jianjizhi" and test usage.    

![/images/2022_09_30_10_30_50_943x660.jpg](/images/2022_09_30_10_30_50_943x660.jpg)

Install intel video:    

![/images/2022_09_30_10_31_52_599x404.jpg](/images/2022_09_30_10_31_52_599x404.jpg)

![/images/2022_09_30_10_32_15_427x337.jpg](/images/2022_09_30_10_32_15_427x337.jpg)

Select `Install this driver software anyway`:     

![/images/2022_09_30_10_32_37_549x382.jpg](/images/2022_09_30_10_32_37_549x382.jpg)

Select `reboot this computer now`:   

![/images/2022_09_30_10_33_49_438x342.jpg](/images/2022_09_30_10_33_49_438x342.jpg)

Status:   

![/images/2022_09_30_10_43_27_657x467.jpg](/images/2022_09_30_10_43_27_657x467.jpg)

