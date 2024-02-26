+++
title= "WorkingTipsOnNvidiaPassthrough3050"
date = "2024-02-26T11:11:14+08:00"
description = "WorkingTipsOnNvidiaPassthrough3050"
keywords = ["Technology"]
categories = ["Technology"]
+++
Edit the grub items:     

```
# vim /etc/default/grub
......
GRUB_CMDLINE_LINUX_DEFAULT="amd_iommu=on iommu=pt kvm.ignore_msrs=1 video=efifb:off"
......
# update-grub2 && reboot
```
update the initramfs:    

```
# vim /etc/initramfs-tools/modules 
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
# update-initramfs -u -k all
```
Download the nvflash, and them dump the vbios:    

```
$ sudo ./nvflash --save vbios.rom
```
patch the vbios.rom:    

Using an hex editor to open the `vbios.rom`(via `bless vbios.rom`):    

![/images/2024_02_26_11_14_14_915x418.jpg](/images/2024_02_26_11_14_14_915x418.jpg)

Search the text `VIDEO`, place your cursor before the first U before the VIDEO we just searched, and select everything before it, delete all.    

![/images/2024_02_26_11_16_26_490x175.jpg](/images/2024_02_26_11_16_26_490x175.jpg)

Select the pci vfio equipments:    

![/images/2024_02_26_11_16_42_258x92.jpg](/images/2024_02_26_11_16_42_258x92.jpg)

romfile should be specified:    

![/images/2024_02_26_11_17_24_731x235.jpg](/images/2024_02_26_11_17_24_731x235.jpg)

```
  <rom file="/usr/share/OVMF/nvidia_vbios.rom"/>
```
Now remove all of the virtual gpu(qxl, virtio-gpu, etc), use nvidia card for boot up. You will get the nvidia card once you installed the nvidia device drivers.    
