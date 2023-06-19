+++
title= "WorkingTipsOnLibvirtBuild"
date = "2023-06-19T09:58:16+08:00"
description = "WorkingTipsOnLibvirtBuild"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Steps
`libvirt-7.5.0`:    

```
yum makecache
yum install -y epel-release
yum install -y meson gcc qemu-kvm wget virt-install python36-docutils python-docutils glib2-devel gnutls-devel libxml2-devel libtirpc-devel
meson --version
0.55.1
wget https://libvirt.org/sources/libvirt-7.5.0.tar.xz --no-check-certificate
tar xJvf libvirt-7.5.0.tar.xz
cd libvirt-7.5.0
meson build --prefix=/usr
ninja -C build
#ninja -C build install
```
`libvirt-8.5.0`:    

```
yum install -y centos-release-scl
yum install devtoolset-11 -y
scl enable devtoolset-11 bash
gcc --version
gcc (GCC) 11.2.1 20220127 (Red Hat 11.2.1-9)
meson build --prefix=/usr
ninja -C build
```
From `8.6.0`:    

```
meson.build:986:0: ERROR: Invalid version of dependency, need 'gnutls' ['>=3.6.0'] found '3.3.29'.
```
From `9.1.0`, Later will have meson version check:    

```
meson.build:1:0: ERROR: Meson version is 0.55.1 but project requires >= 0.56.0
```

### product env
build and install via:      

```
yum install -y gnutls-devel
yum install -y python36-docutils python-docutils glib2-devel gnutls-devel libxml2-devel libtirpc-devel
meson build --prefix=/opt/local/
ninja -C build
ninja -C build install
```
Change configuration files:     

```
vim /usr/lib/systemd/system/libvirtd.servic
/usr/bin/libvirtd --> /opt/local/bin/libvirtd
reboot
/opt/local/bin/virsh  list --all
```
### Rebuild
Rebuild with full support:    

```
yum install -y netcf-devel libpciaccess-devel systemd-devel
meson build -Dsystem=true -Ddriver_qemu=enabled -Ddriver_interface=enabled -Ddriver_libvirtd=enabled -Ddriver_remote=enabled -Ddriver_network=enabled --prefix=/usr
ninja -C build install
```
Edit the qemu configuration:    

```
# vim /etc/libvirt/qemu.conf
cgroup_device_acl = [
    "/dev/null", "/dev/full", "/dev/zero",
    "/dev/random", "/dev/urandom",
    "/dev/ptmx", "/dev/kvm", "/dev/udmabuf"
]
user = "root"
group = "root"
```
