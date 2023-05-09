+++
title= "Gen12GPUPassthrough"
date = "2023-05-08T09:28:47+08:00"
description = "Gen12GPUPassthrough"
keywords = ["Technology"]
categories = ["Technology"]
+++
`Ubuntu 22.04.2` desktop cdrom installation on Gen12 machine.   

```
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y openssh-server virt-manager qemu vim
```
Edit the grub and initramfs-tools, and vfio:    

```
$ cat /etc/default/grub | grep CMDLINE_LINUX_DEFAULT
GRUB_CMDLINE_LINUX_DEFAULT="quiet  intel_iommu=on intel_iommu=pt kvm.ignore_msrs=1 video=efifb:off,vesafb:off"
$ cat /etc/initramfs-tools/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
$  echo "options vfio-pci disable_vga=1" > /etc/modprobe.d/vfio.conf
$ sudo update-grub2 && sudo update-initramfs -u -k all
$ sudo reboot
```
Build ovmf:    

```
$ cp /home/idv/xxxxxxxxx/VBT/Vbt.bin OvmfPkg/Vbt/Vbt.bin 
$ cp /home/idv/xxxxxxxxx/./IntelGopDriver/RELEASE_VS2015x86/X64/IntelGopDriver.efi OvmfPkg/IntelGop/IntelGopDriver.efi
$ pwd
/home/idv/Code/Intel_edk2/edk2_gen12
Build the ovmf(follow the guideline)
```

Install an ubuntu22.04 vm in host machine, with qxl/spice.      

glmark2: score 1711

Install win11(shift+f10), use regedit for editing:    

![/images/2023_05_08_11_23_09_794x480.jpg](/images/2023_05_08_11_23_09_794x480.jpg)

Fallback and continue install:    

![/images/2023_05_08_11_24_16_769x553.jpg](/images/2023_05_08_11_24_16_769x553.jpg)
 
