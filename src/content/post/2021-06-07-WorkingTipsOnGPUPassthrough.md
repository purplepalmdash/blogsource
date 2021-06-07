+++
title= "WorkingTipsOnGPUPassthrough"
date = "2021-06-07T07:44:45+08:00"
description = "WorkingTipsOnGPUPassthrough"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Hardware Info
gpu info:    

```
root@usbserver:~# lspci | grep -i nvidia
01:00.0 VGA compatible controller: NVIDIA Corporation GP106M [GeForce GTX 1060 Mobile] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GP106 High Definition Audio Controller (rev a1)
root@usbserver:~# uname -a
Linux usbserver 5.4.0-74-generic #83-Ubuntu SMP Sat May 8 02:35:39 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
root@usbserver:~# cat /etc/issue
Ubuntu 20.04.2 LTS \n \l
```
Newly installed system, install qemu/libvirtd/virt-manager via:    

```
# apt-get update -y && apt-get install -y virt-manager
```

### Configuration
Remove current NVIDIA or opensource Nouveau driver:    

```
root@usbserver:~# apt-get remove --purge nvidia*
```
Add driver blacklist:    

```
# vim /etc/modprobe.d/blacklist.conf
....
blacklist amd76x_edac #this might not be required for x86 32 bit users.
blacklist vga16fb
blacklist nouveau
blacklist rivafb
blacklist nvidiafb
blacklist rivatv
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
```
Disable modset and add `intel_iommu` for default grub setting and update grub:    

```
# vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="maybe-ubiquity nouveau.modeset=0 intel_iommu=on"
# update-grub2
```
Disable nouveau kernel:    

```
# echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf
# update-initramfs -u
```
On the above output for `lspci -nn | grep -i nvidia`, record the GPU id `10de:1c20` and `10de:10f1`, create `/etc/modprobe.d/vfio.conf`, add corresponding GPU PCI Ids:    

```
# vim /etc/modprobe.d/vfio.conf
options vfio-pci ids=10de:10f1,10de:1c20
```
Edit `/etc/default/grub` for adding `vfio-ids`:    

```
# vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="maybe-ubiquity nouveau.modeset=0 intel_iommu=on vfio-pci.ids=10de:1c20,10de:10f1"
# update-grub2
```
Add vfio modules into initrd:    

```
# vim /etc/initramfs-tools/modules

vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
# update-initramfs -u
```
Now restart host machine, after boot check configurations for vfio:    

```
# dmesg | grep -i iommu
# dmesg | grep -i dmar
# lspci -vv
See if the  drive in use is vfio-pci
```
### VM

