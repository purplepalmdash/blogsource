+++
title= "WorkingTipsOnGen12CentOS"
date = "2023-06-07T08:53:19+08:00"
description = "WorkingTipsOnGen12CentOS"
keywords = ["Technology"]
categories = ["Technology"]
+++
CentOS7.6.1810,minimum installation.    

```
# yum makecache
# yum install -y vim git gcc libevent-devel unzip flex bison
# vim /etc/selinux/config
SELINUX=disabled
# systemctl disable firewalld
```
Install gcc-7:     

```
yum install centos-release-scl
yum install devtoolset-7-gcc-c++
scl enable devtoolset-7 bash
```
Install kernel related dependencies:     

```
yum install -y devtoolset-7-elfutils-libelf-devel.x86_64  openssl-devel bc rpm-build pciutils

```

Clone the kernel code:    

```
# mkdir Code
# cd Code
# git clone https://github.com/intel/linux-intel-lts.git
# cd linux-intel-lts
# git checkout gawougoweugowugo
# cp ~/kernel-config.zip .
# unzip kernel-config.zip
# cp ./kernel-config/x86_64_defconfig .config
# echo "" | make ARCH=x86_64 olddefconfig
# make ARCH=x86_64 -j16 LOCALVERSION=-lts2021-iotg -j8
```

### Verification
Make sure you have the latest firmware(copy from the ubuntu):    

```
# mkdir /lib/firmware/i915/backup
# mv /lib/firmware/i915/* /lib/firmware/i915/backup
# cp -ar /mnt/lib/firmware/i915/* /lib/firmware/i915/
```
Install the minimal system, install the built kernel rpms, then edit the grub:    

```
# vim /etc/default/grub
......
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet i915.enable_guc=0x7 udmabuf.list_limit=8192 intel_iommu=on i915.force_probe=*"
......
# grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
# cat /boot/grub2/grubenv 
    # GRUB Environment Block
    #saved_entry=CentOS Linux (3.10.0-957.el7.x86_64) 7 (Core)
    saved_entry=CentOS Linux (5.15.85+) 7 (Core)
# reboot
```
Check the status of `SR-IOV`:   

```
# dmesg | grep SR-IOV
[    1.643471] i915 0000:00:02.0: Running in SR-IOV PF mode
# dmesg | grep GuC
[    1.844856] i915 0000:00:02.0: [drm] GuC error state capture buffer maybe too small: 2097152 < 2557128 (min = 852376)
[    1.846949] i915 0000:00:02.0: [drm] GuC firmware i915/tgl_guc_70.bin version 70.5.1
[    1.850099] i915 0000:00:02.0: [drm] GuC submission enabled
[    1.850099] i915 0000:00:02.0: [drm] GuC SLPC enabled
[    1.850430] i915 0000:00:02.0: [drm] GuC RC: enabled
# dmesg | grep HuC
[    1.846952] i915 0000:00:02.0: [drm] HuC firmware i915/tgl_huc.bin version 7.9.3
[    1.849623] i915 0000:00:02.0: [drm] HuC authenticated
```
Install necessary packages:     

```
# yum install -y virt-manager
# yum groupinstall "GNOME Desktop"
```

Upgrade qemu:    

```
# yum install centos-release-qemu-ev
# yum makecache
# yum install -y qemu-kvm-ev qemu-img-ev
# /usr/libexec/qemu-kvm --version
QEMU emulator version 2.12.0 (qemu-kvm-ev-2.12.0-44.1.el7_8.1)
Copyright (c) 2003-2017 Fabrice Bellard and the QEMU Project developers
```
Create vf:    

```
[root@cs76 ~]# lspci | grep -i vga
0000:00:02.0 VGA compatible controller: Intel Corporation Device 4680 (rev 0c)
[root@cs76 ~]# chmod 777 createvf.sh 
.[root@cs76 ~]# ./createvf.sh 
0x8086 0x4680
[root@cs76 ~]# lspci | grep -i vga
0000:00:02.0 VGA compatible controller: Intel Corporation Device 4680 (rev 0c)
0000:00:02.1 VGA compatible controller: Intel Corporation Device 4680 (rev 0c)
0000:00:02.2 VGA compatible controller: Intel Corporation Device 4680 (rev 0c)
0000:00:02.3 VGA compatible controller: Intel Corporation Device 4680 (rev 0c)
0000:00:02.4 VGA compatible controller: Intel Corporation Device 4680 (rev 0c)
0000:00:02.5 VGA compatible controller: Intel Corporation Device 4680 (rev 0c)
0000:00:02.6 VGA compatible controller: Intel Corporation Device 4680 (rev 0c)
0000:00:02.7 VGA compatible controller: Intel Corporation Device 4680 (rev 0c)
```
Install some management tools:    

```
yum install libvirt libvirt-python libguestfs-tools virt-install -y
```

### switch to qemu 6.2
build ninja:    

```
yum install -y gcc gcc-c++ python3 git
git clone https://github.com/ninja-build/ninja.git
cd ninja
git checkout release
./configure.py --bootstrap
cp ninja /usr/bin
```
Install qemu build dependencies:     

```
yum install -y glib2-devel pixman-devel zlib-devel libusb-devel libusb libusbx-devel pulseaudio-libs-devel libcap-ng-devel libattr-devel spice-server-devel usbredir-devel
```
Since qemu 6.2 requires gcc>7.6? Install gcc 8 for building:    

```
yum install devtoolset-8-gcc-c++
scl enable devtoolset-8 bash
./configure --target-list=x86_64-softmmu --enable-debug --disable-docs --disable-virglrenderer --prefix=/opt/local --enable-virtfs --enable-libusb --disable-debug-tcg --audio-drv-list=pa  --enable-spice --enable-usb-redir
make -j8
make install
```
Change from default qemu to `/opt/local/bin/qemu-system-x86_64`, then start the machine.   
You may also need virsh or other tools:    

```
yum install -y libvirt libvirt-python libguestfs-tools virt-install -y
```
