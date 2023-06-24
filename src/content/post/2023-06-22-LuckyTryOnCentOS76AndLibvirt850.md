+++
title= "LuckyTryOnCentOS76AndLibvirt850"
date = "2023-06-22T08:45:30+08:00"
description = "LuckyTryOnCentOS76AndLibvirt850"
keywords = ["Technology"]
categories = ["Technology"]
+++
steps:    

```
[root@ttt yum.repos.d]# cat local.repo 
[local]
name=local
baseurl=file:///root/rpms
enabled=1
gpgcheck=0
# yum makecache
```

### Using vm
using vm(nested kvm) for trying.   

![/images/2023_06_24_07_46_02_1050x717.jpg](/images/2023_06_24_07_46_02_1050x717.jpg)

View the default installation version:    

```
[root@lucky ~]# /usr/libexec/qemu-kvm  --version
QEMU emulator version 1.5.3 (qemu-kvm-1.5.3-160.el7), Copyright (c) 2003-2008 Fabrice Bellard
[root@lucky ~]# libvirtd --version
libvirtd (libvirt) 4.5.0
```
Install kernel and changes to new kernel:    

```
# rpm -ivh kernel-5.15.85_lts2021_kkkk-3.x86_64.rpm 
Preparing...                          ################################# [100%]
Updating / installing...
 1:kernel-5.15.85_lts2021_kkkk-3    ################################# [100%]
# vi /boot/grub2/grubenv
saved_entry=CentOS Linux (5.15.85-lts2021-kkkk) 7 (Core)
```
Change the selinux and firewalld:    

```
[root@lucky ~]# systemctl disable firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
[root@lucky ~]# vim /etc/selinux/config 
```
Install `gcc-11` for building libvirtd and qemu:    

```
yum install -y centos-release-scl
yum install -y devtoolset-11 bc
```
Build Qemu:     

```
# mkdir Code
# mv libvirt-8.5.0.tar.xz qemu_spice_modified.tar.gz Code
# cd Code
# tar xJvf libvirt-8.5.0.tar.xz 
# tar xzvf qemu_spice_modified.tar.gz 
# which ninja
# yum install -y git glib2-devel pixman-devel zlib-devel libusb-devel libusb libusbx-devel pulseaudio-libs-devel libcap-ng-devel libattr-devel spice-server-devel usbredir-devel python3 bzip2
# cd qemu-7.1.0
# scl enable devtoolset-11 bash
#  ./configure --enable-modules --target-list=x86_64-softmmu --enable-debug --disable-docs --disable-virglrenderer --prefix=/opt/local --enable-virtfs --enable-libusb --disable-debug-tcg --audio-drv-list=pa
# make -j8
# make install
# /opt/local/bin/qemu-system-x86_64 --version
QEMU emulator version 7.1.0
Copyright (c) 2003-2022 Fabrice Bellard and the QEMU Project developers
```
Build libvirtd:    

```
# yum install -y epel-release
# yum install -y meson python36-docutils python-docutils glib2-devel gnutls-devel libxml2-devel libtirpc-devel gnutls-devel netcf-devel libpciaccess-devel systemd-devel yajl-devel
Build to /usr
and install
```
