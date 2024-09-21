+++
title= "RebuildQemuOnDebian"
date = "2024-09-21T09:50:17+08:00"
description = "RebuildQemuOnDebian"
keywords = ["Technology"]
categories = ["Technology"]
+++
Steps:   

```
$ vim /etc/apt/sources.list
Add:  
 deb http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware
 deb-src http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware
$ sudo apt build-dep qemu-system
$ mkdir -p ~/qemu && cd qemu
$ apt source qemu-system
$ cd qemu-9.0.2+ds
$ dpkg-buildpackage -rfakeroot -uc -b
```
After building, the debs is listed as:    

```
test@buildqemu:~/qemu$ ls
qemu-9.0.2+ds                                           qemu-system-misc-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu_9.0.2+ds-1~bpo12+1_amd64.buildinfo                 qemu-system-modules-opengl_9.0.2+ds-1~bpo12+1_amd64.deb
qemu_9.0.2+ds-1~bpo12+1_amd64.changes                   qemu-system-modules-opengl-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu_9.0.2+ds-1~bpo12+1.debian.tar.xz                   qemu-system-modules-spice_9.0.2+ds-1~bpo12+1_amd64.deb
qemu_9.0.2+ds-1~bpo12+1.dsc                             qemu-system-modules-spice-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu_9.0.2+ds.orig.tar.xz                               qemu-system-ppc_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-block-extra_9.0.2+ds-1~bpo12+1_amd64.deb           qemu-system-ppc-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-block-extra-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb    qemu-system-sparc_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-guest-agent_9.0.2+ds-1~bpo12+1_amd64.deb           qemu-system-sparc-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-guest-agent-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb    qemu-system-x86_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system_9.0.2+ds-1~bpo12+1_amd64.deb                qemu-system-x86-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-arm_9.0.2+ds-1~bpo12+1_amd64.deb            qemu-system-xen_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-arm-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb     qemu-system-xen-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-common_9.0.2+ds-1~bpo12+1_amd64.deb         qemu-user_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-common-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb  qemu-user-binfmt_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-data_9.0.2+ds-1~bpo12+1_all.deb             qemu-user-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-gui_9.0.2+ds-1~bpo12+1_amd64.deb            qemu-user-static_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-gui-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb     qemu-user-static-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-mips_9.0.2+ds-1~bpo12+1_amd64.deb           qemu-utils_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-mips-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb    qemu-utils-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-misc_9.0.2+ds-1~bpo12+1_amd64.deb

```
