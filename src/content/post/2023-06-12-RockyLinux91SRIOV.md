+++
title= "RockyLinux91SRIOV"
date = "2023-06-12T11:56:30+08:00"
description = "RockyLinux91SRIOV"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Kernel
Install:    

```
tar xzvf linux-intel-lts.tar.gz
yum install -y gcc openssl-devel bc rpm-build pciutils flex bison elfutils-libelf-devel
cd linux-intel-lts
make distclean
cp ./kernel-config/x86_64_defconfig .config
echo "" | make ARCH=x86_64 olddefconfig
update-crypto-policies --set LEGACY
reboot
make rpm-pkg -j8
```
Replace the repository:   

```
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/rocky-extras.repo \
    /etc/yum.repos.d/rocky.repo
yum makecache
```
Install kernel:    

```
cp i915/* /lib/firmware/i915/
rpm -ivh kernel-5.15.85-1.x86_64.rpm kernel-headers-5.15.85-1.x86_64.rpm
vi /etc/default/grub
Add "i915.enable_guc=0x7 udmabuf.list_limit=8192 intel_iommu=on i915.force_probe=*" to 
grub2-mkconfig -o /boot/efi/EFI/rocky/grub.cfg
reboot
```
Examine via:    

```
[root@localhost ~]# uname -r
5.15.85
[root@localhost ~]# dmesg | grep GuC
[    5.176714] i915 0000:00:02.0: [drm] GuC error state capture buffer maybe too small: 2097152 < 2163708 (min = 721236)
[    5.179124] i915 0000:00:02.0: [drm] GuC firmware i915/tgl_guc_70.bin version 70.5.1
[    5.182137] i915 0000:00:02.0: [drm] GuC submission enabled
[    5.182138] i915 0000:00:02.0: [drm] GuC SLPC enabled
[    5.182455] i915 0000:00:02.0: [drm] GuC RC: enabled
[root@localhost ~]# dmesg | grep HuC
[    5.179129] i915 0000:00:02.0: [drm] HuC firmware i915/tgl_huc.bin version 7.9.3
[    5.181869] i915 0000:00:02.0: [drm] HuC authenticated
[root@localhost ~]# dmesg | grep SR-IOV
[    5.060282] i915 0000:00:02.0: Running in SR-IOV PF mode
```
### Qemu/libvirt/virt-manager
Install following packages for using qemu:    

```
# yum install -y virt-manager qemu-kvm
# systemctl enable libvirtd
# systemctl start libvirtd
# /usr/libexec/qemu-kvm --version
QEMU emulator version 7.2.0 (qemu-kvm-7.2.0-14.el9_2)
Copyright (c) 2003-2022 Fabrice Bellard and the QEMU Project developers
# yum groupinstall -y "Server with GUI"
```
