+++
title= "WorkingTipsOn3060vfioPassthrough"
date = "2024-01-19T11:12:04+08:00"
description = "WorkingTipsOn3060vfioPassthrough"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Host Preparation
Edit grub configuration:    

```
# vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on iommu=pt"

# update-grub
```
Edit vfio items:    

```
# vim /etc/modprobe.d/vfio.conf
options vfio-pci ids=10de:2508

```
Edit initramfs items:    

```
# vim /etc/initramfs-tools/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
Bios configuration, disable igpu, and select `PEG`:    

```
PEG is technically just "PCI Express Graphics" your 16x lane slot to the CPU.
```
Add kvm items:    

```
# vim /etc/modprobe.d/kvm.conf
options kvm ignore_msrs=1
```
Get the device via devicehunt.com:     

```
GA106 [GeForce RTX 3050 OEM]
Type	Information
ID	2508
Vendor Details
NVIDIA Corporation
Type	Information
ID	10DE
```
Use gpu-z under windows to fetch the rom.  

Edit the file using `bless` under linux, find `55AA`:     

![/images/2024_01_19_15_02_58_1227x693.jpg](/images/2024_01_19_15_02_58_1227x693.jpg)

Delete everything before this `55AA` :      

![/images/2024_01_19_15_05_02_1248x606.jpg](/images/2024_01_19_15_05_02_1248x606.jpg)

Create the qcow2, specify its backup file:    

```
qemu-img create -f qcow2 -b win10_pure_with_rdp_open.qcow2 -F qcow2 win10_nvidia_3050.qcow2
```
Create the machine:    

![/images/2024_01_19_15_30_00_494x489.jpg](/images/2024_01_19_15_30_00_494x489.jpg)

Specify its cpus and memory:    

![/images/2024_01_19_15_30_26_424x168.jpg](/images/2024_01_19_15_30_26_424x168.jpg)

Specify its name:     

![/images/2024_01_19_15_31_04_495x322.jpg](/images/2024_01_19_15_31_04_495x322.jpg)

Custom its chipset and firmware:     

![/images/2024_01_19_15_31_31_572x194.jpg](/images/2024_01_19_15_31_31_572x194.jpg)

Add a new network(macvtap) for rdp:     

![/images/2024_01_19_15_32_24_570x305.jpg](/images/2024_01_19_15_32_24_570x305.jpg)

Using qxl and spice for verifying its working, then add nvidia 3050.    

### Adding GPU items
Getting the gpu card related:   

```
$ sudo lspci -nn  | grep 03:00
03:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:2508] (rev a1)
03:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:228e] (rev a1)
```
Verify its iommu groups:     

```
$ for a in /sys/kernel/iommu_groups/*; do find $a -type l; done | sort --version-sort
......

/sys/kernel/iommu_groups/14/devices/0000:03:00.0
/sys/kernel/iommu_groups/14/devices/0000:03:00.1

......
```
Edit the grub parameters:     

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on iommu=pt kvm.ignore_msrs=1 vfio-pci.ids=10de:2508,10de:228e"
```
update grub then reboot. verify kernel driver in use via:    

```
# lspci -kn | grep -A 2 03:00
03:00.0 0300: 10de:2508 (rev a1)
	Subsystem: 1028:c97c
	Kernel driver in use: vfio-pci
--
03:00.1 0403: 10de:228e (rev a1)
	Subsystem: 1028:c97c
	Kernel driver in use: vfio-pci
```    
Add the pci devices :    

![/images/2024_01_19_15_45_32_836x497.jpg](/images/2024_01_19_15_45_32_836x497.jpg)

The modified content is listed as :    

```
<hostdev mode="subsystem" type="pci" managed="yes">
  <source>
    <address domain="0x0000" bus="0x03" slot="0x00" function="0x0"/>
  </source>
  <rom file="/usr/share/OVMF/GA106.rom"/>
  <address type="pci" domain="0x0000" bus="0x06" slot="0x00" function="0x0" multifunction="on"/>
</hostdev>

```

将显卡从第二槽位到第一槽位，解决了vfio问题。   

![/images/2024_01_19_16_26_23_499x188.jpg](/images/2024_01_19_16_26_23_499x188.jpg)

![/images/2024_01_19_16_29_16_590x433.jpg](/images/2024_01_19_16_29_16_590x433.jpg)

添加usb设备:    

![/images/2024_01_19_16_34_00_227x132.jpg](/images/2024_01_19_16_34_00_227x132.jpg)

去掉qxl及spice, 只使用实际的物理显卡设备来进行测试. 成功，但是uefi启动的时候无tiano core的画面。     
