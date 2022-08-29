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
