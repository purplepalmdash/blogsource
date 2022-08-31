+++
title= "WorkingTipsOnUpgradingSG1DriverCentOS1810.qcow2"
date = "2022-08-29T10:25:20+08:00"
description = "WorkingTipsOnUpgradingSG1DriverCentOS1810.qcow2"
keywords = ["Technology"]
categories = ["Technology"]
+++
### VM
Import new image:    

![/images/2022_08_29_10_25_29_498x422.jpg](/images/2022_08_29_10_25_29_498x422.jpg)

Name it:    

![/images/2022_08_29_10_26_09_467x332.jpg](/images/2022_08_29_10_26_09_467x332.jpg)

Upgrade kernel to 4.19.12:    

```
$ ls
kernel-4.19.12-1.x86_64.rpm  kernel-devel-4.19.12-1.x86_64.rpm  kernel-headers-4.19.12-1.x86_64.rpm  kmod-ukmd-4.19.12-20212.el7.x86_64.rpm
$ rpm -ivh *.rpm
$ vi /etc/default/grub
Changed to 4.19.12 kernel
$ grub2-mkconfig -o /boot/grub2/grub.cfg 
$ sudo reboot
```
Change the repository using iso:    

```
# mount /dev/sr0 /mnt
# cat /etc/yum.repos.d/local.repo
[local]
name=local
baseurl=file:///mnt
enabled=1
gpgcheck=0
# yum makecache
```
Using official `install-sg1.sh` for installing to 4.19.112 kernel for fetching the dependencies.  

### Merge steps
安装官方驱动的过程中会编译出相应的文件，需要按照以下步骤进行更改:     

```
# mkdir /root/merge
# cp /root/rpmbuild/SRPMS/kmod-ukmd-4.19.112-20413.el7.centos.src.rpm /root/merge
# cd /root/merge
# mkdir kmod-ukmd-4.19.12.cpio
# cd kmod-ukmd-4.19.12.cpio
# rpm2cpio ../kmod-ukmd-4.19.112-20413.el7.centos.src.rpm | cpio -idmv
# ls
dg1_dmc_ver2_02.bin  dg1_guc_49.0.1.bin  dg1_huc_7.9.5.bin  kmod.spec  kmod-ukmd-4.19.112.tar.gz
# tar xzvf kmod-ukmd-4.19.112.tar.gz
# mv kmod-ukmd-4.19.112 kmod-ukmd-4.19.12
# vim kmod.spec 
Change 4.19.112->4.19.12
# vim kmod-ukmd-4.19.12/kmod.spec
Change 4.19.112->4.19.12
# cp /usr/src/kernels/4.19.12/drivers/gpu/drm/drm_edid.c kmod-ukmd-4.19.12/orig/drivers/gpu/drm/
# cp /usr/src/kernels/4.19.12/drivers/gpu/drm/drm_probe_helper.c kmod-ukmd-4.19.12/orig/drivers/gpu/drm/
# cp /usr/src/kernels/4.19.12/drivers/gpu/drm/drm_vblank.c kmod-ukmd-4.19.12/orig/drivers/gpu/drm/
# mv kmod-ukmd-4.19.112.tar.gz /tmp
# tar czvf kmod-ukmd-4.19.12.tar.gz kmod-ukmd-4.19.12
# rm -rf kmod-ukmd-4.19.12
# ls
dg1_dmc_ver2_02.bin  dg1_guc_49.0.1.bin  dg1_huc_7.9.5.bin  kmod.spec  kmod-ukmd-4.19.12.tar.gz
# cd ..
# tar czvf kmod-ukmd-4.19.12.cpio.tar.gz kmod-ukmd-4.19.12.cpio
# ls
kmod-ukmd-4.19.112-20413.el7.centos.src.rpm  kmod-ukmd-4.19.12.cpio  kmod-ukmd-4.19.12.cpio.tar.gz
# mv /root/rpmbuild/SOURCES/ /root/rpmbuild/SOURCES.back && mkdir -p /root/rpmbuild/SOURCES/
# cp -ar /root/merge/kmod-ukmd-4.19.12.cpio/* /root/rpmbuild/SOURCES
# cd /root/rpmbuild/SOURCES
# vim /usr/src/kernels/4.19.12/include/linux/math64.h
在文件末尾倒数第二行添加
.....
    #endif /* mul_u64_u32_div */
    +++++ #define DIV64_U64_ROUND_UP(ll, d)	\
    +++++	({ u64 _tmp = (d); div64_u64((ll) + _tmp - 1, _tmp); })
    
    #endif /* _LINUX_MATH64_H */

# rpmbuild -bb kmod.spec
# ls /root/rpmbuild/RPMS/x86_64
kmod-ukmd-4.19.12-20413.el7.centos.x86_64.rpm
``` 

### Verification
Install any 4.19.12 kernel, test the package  `kmod-ukmd-4.19.12-20413.el7.centos.x86_64.rpm`.     

```
$ ls /dev/dri/
card0  renderD128
$ lsmod | grep i915
i915_spi               20480  0 
mtd                    57344  5 i915_spi
i915                 1933312  0 
drm_ukmd_kms_helper   188416  1 i915
drm_ukmd              495616  3 drm_ukmd_kms_helper,i915
drm_ukmd_compat        98304  3 drm_ukmd_kms_helper,i915,drm_ukmd
video                  40960  1 i915
i2c_algo_bit           16384  1 i915
cec                    45056  1 i915
mfd_core               16384  2 lpc_ich,i915
```
Examine the kernel demsg:    

```
$ sudo dmesg | grep drm
[sudo] password for ctctest: 
[    7.776406] drm: loading out-of-tree module taints kernel.
[    7.800836] Initialized drm/i915 compat module 20160105
[    7.892916] [drm] i915 backporting
[    7.893701] [drm] Intel graphics LMEM: [mem 0x00000000-0x1fb7fffff]
[    7.893702] [drm] Intel graphics LMEM IO start: a00000000
[    7.893702] [drm] Intel graphics LMEM size: 1fb800000
[    7.893714] [drm] Intel graphics stolen LMEM: [mem 0x1fc000000-0x1ffffffff]
[    7.893714] [drm] Intel graphics stolen LMEM IO start: bfc000000
[    7.895015] i915 0000:07:00.0: [drm] Couldn't get system memory bandwidth
[    7.895053] [drm] Supports vblank timestamp caching Rev 2 (21.10.2013).
[    7.895054] [drm] Driver supports precise vblank timestamp query.
[    7.971850] i915 0000:07:00.0: [drm] Finished loading DMC firmware i915/dg1_dmc_ver2_02.bin (v2.2)
[    7.994442] [drm] GuC communication enabled
[    7.999403] i915 0000:07:00.0: [drm] GuC firmware i915/dg1_guc_49.0.1.bin version 49.0 submission:disabled
[    7.999405] i915 0000:07:00.0: [drm] HuC firmware i915/dg1_huc_7.9.5.bin version 7.9 authenticated:yes
[    8.001958] [drm] Initialized i915 1.6.0 20220718-8a58be7 for 0000:07:00.0 on minor 0
[    8.003950] i915 0000:07:00.0: [drm] Cannot find any crtc or sizes
```

