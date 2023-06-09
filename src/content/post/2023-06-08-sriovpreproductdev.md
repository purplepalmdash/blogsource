+++
title= "sriovpreproductdev"
date = "2023-06-08T14:42:41+08:00"
description = "sriovpreproductdev"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Kernel
CentOS 7.6.1810, minimal installtion.     

Enable keep cache:    

```
# vi /etc/yum.conf
...
keepcache=1
...
```
Replace i915 firmware:    

```
mv /lib/firmware/i915/ /lib/firmware/i915.back
tar xzvf materials/i915.tar.gz -C /lib/firmware/
```
Install Kernel:    

```
# rpm -ivh materials/kernel-5.15.85+-1.x86_64.rpm
# vi /boot/grub2/grubenv
saved_entry=CentOS Linux (5.15.85+) 7 (Core)
# vi /etc/default/grub
Add "i915.enable_guc=0x7 udmabuf.list_limit=8192 intel_iommu=on i915.force_probe=*" to 
GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet i915.enable_guc=0x7 udmabuf.list_limit=8192 intel_iommu=on i915.force_probe=*"
# grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
# reboot
```
Check the status:    

```
[root@i3 ~]# dmesg | grep SR-IOV
[    1.025680] i915 0000:00:02.0: Running in SR-IOV PF mode
[root@i3 ~]# dmesg | grep GuC
[    1.157064] i915 0000:00:02.0: [drm] GuC error state capture buffer maybe too small: 2097152 < 2163708 (min = 721236)
[    1.161653] i915 0000:00:02.0: [drm] GuC firmware i915/tgl_guc_70.bin version 70.5.1
[    1.166302] i915 0000:00:02.0: [drm] GuC submission enabled
[    1.166304] i915 0000:00:02.0: [drm] GuC SLPC enabled
[    1.166591] i915 0000:00:02.0: [drm] GuC RC: enabled
[root@i3 ~]# dmesg | grep HuC
[    1.161660] i915 0000:00:02.0: [drm] HuC firmware i915/tgl_huc.bin version 7.9.3
[    1.166031] i915 0000:00:02.0: [drm] HuC authenticated
```
### Qemu
Install following packages:   

```
# yum install -y libvirt libvirt-python libguestfs-tools virt-install pciutils gcc gcc-c++ python3 git glib2-devel pixman-devel zlib-devel libusb-devel libusb libusbx-devel pulseaudio-libs-devel libcap-ng-devel libattr-devel spice-server-devel usbredir-devel centos-release-scl  unzip OVMF
# yum install -y devtoolset-8-gcc-c++
# cp materials/ninja /usr/bin
# chmod 777 /usr/bin/ninja 
```
Build Qemu:    

```
# tar xJvf qemu-6.2.0.tar.xz
# scl enable devtoolset-8 bash
# mkdir /opt/local
# cd qemu-6.2.0
# scl enable devtoolset-8 bash
# ./configure --target-list=x86_64-softmmu --enable-debug --disable-docs --disable-virglrenderer --prefix=/opt/local --enable-virtfs --enable-libusb --disable-debug-tcg --audio-drv-list=pa  --enable-spice --enable-usb-redir
# make -j8
# make install
# /opt/local/bin/qemu-system-x86_64 --version
QEMU emulator version 6.2.0
Copyright (c) 2003-2021 Fabrice Bellard and the QEMU Project developers
```
start and enable libvirtd :    

```
# systemctl enable libvirtd
# systemctl start libvirrtd
```
Add following lines to `/etc/rc.local`:    

```
## start of create vf
sudo modprobe i2c-algo-bit
sudo modprobe video
echo '0' | sudo tee -a /sys/bus/pci/devices/0000\:00\:02.0/sriov_drivers_autoprobe > /dev/null
echo 7 | sudo tee -a /sys/class/drm/card0/device/sriov_numvfs > /dev/null
echo '1' | sudo tee -a /sys/bus/pci/devices/0000\:00\:02.0/sriov_drivers_autoprobe > /dev/null
sudo modprobe vfio-pci
vendor=$(cat /sys/bus/pci/devices/0000:00:02.0/iommu_group/devices/0000:00:02.0/vendor)
device=$(cat /sys/bus/pci/devices/0000:00:02.0/iommu_group/devices/0000:00:02.0/device)
echo $vendor $device  | sudo tee -a /sys/bus/pci/drivers/vfio-pci/new_id
## endof create vf
```
### Create virtual machine
Define win10 machine:    

```
virsh define materials/wwin10.xml
```
Transfer the image to machine:    

```
# scp /var/lib/libvirt/images/winpure.qcow2 root@192.168.1.120:/var/lib/libvirt/images/
```
Failed, no display output.  
### r8168
Get the driver source code from:   `https://www.realtek.com/en/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software`,     

```
rpm -ivh kernel-devel-5.15.85+-1.x86_64.rpm
yum remove kernel-headers
rpm -ivh kernel-headers-5.15.85+-1.x86_64.rpm
yum install -y centos-release-scl
yum install devtoolset-7
scl enable devtoolset-7 bash
cd r8168-8.051.02/src/
make
modprobe r8168
```
Add `modprobe r8168` to `/etc/rc.local
