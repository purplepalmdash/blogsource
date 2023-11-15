+++
title= "WorkingTipsOnBackPortKernel"
date = "2023-11-15T08:57:43+08:00"
description = "WorkingTipsOnBackPortKernel"
keywords = ["Technology"]
categories = ["Technology"]
+++
Configure repository and install packages:   

```
yum install -y make glibc-devel rpm-build bison flex gawk
dnf config-manager --set-enabled crb
dnf install epel-release
yum install -y dkms
```
Set the default boot kernel:    

```
# grubby --set-default /boot/vmlinuz-5.14.0-284.30.1.el9_2.x86_64 
# reboot

verification
# uname -r
5.14.0-284.30.1.el9_2.x86_64
```
Make dkms packages:     

```
cd /root/intel-gpu-i915-backports
vim versions
...
RHEL_9.2_KERNEL_VERSION="5.14.284-1_rc1"
...
make i915dkmsrpm-pkg OS_DISTRIBUTION=RHEL_9.2
ls -l -h /root/rpmbuild/RPMS/x86_64/intel-i915-dkms-1.23.7.17.230608.25.5.14.284.1_rc1-1.x86_64.rpm
-rw-r--r--. 1 root root 2.7M Nov 15 03:02 /root/rpmbuild/RPMS/x86_64/intel-i915-dkms-1.23.7.17.230608.25.5.14.284.1_rc1-1.x86_64.rpm
```
