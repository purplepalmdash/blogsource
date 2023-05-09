+++
title= "idvworkingtips2"
date = "2023-03-28T14:11:55+08:00"
description = "idvworkingtips2"
keywords = ["Technology"]
categories = ["Technology"]
+++
### System Installation
`Ubuntu 22.04.2 iso`, Install with HWE Kernel Selected.   
English-> Ubuntu Server(mimimized) -> Skip Ubuntu Pro -> Install OpenSSH server.   

```
sudo apt update
sudo apt install -y vim sddm libvirt-daemon qemu libvirt-daemon-system-systemd libvirt-daemon-driver-qemu libvirt-daemon-system libvirt0
```
Examine the version:    

```
$ qemu-system-x86_64 --version
QEMU emulator version 6.2.0 (Debian 1:6.2+dfsg-2ubuntu6.6)
Copyright (c) 2003-2021 Fabrice Bellard and the QEMU Project developers
```
### System Configuration
Kernel and modules:    

```
# sudo vim /etc/default/grub
......
GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on intel_iommu=pt kvm.ignore_msrs=1"
......
# sudo update-grub2
# sudo vim /etc/initramfs-tools/modules
......
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
# update-initramfs -u -k all
# sudo reboot
```
Use mate for desktop  session:    

```
$ sudo apt install -y mate
```
libvirt hook should be the same.    

sddm autologin :    

```
# mkdir -p /etc/sddm.conf.d
# vim /etc/sddm.conf.d/autologin.conf
[Autologin]
User=idv
Session=mate

```
while the X Session could be viewd as:    

```
l /usr/share/xsessions/
ubuntu.desktop  ubuntu-xorg.desktop

```
