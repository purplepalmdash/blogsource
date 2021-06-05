+++
title= "WorkingTipsOnGpuPassthrough"
date = "2021-06-05T08:01:14+08:00"
description = "WorkingTipsOnGpuPassthrough"
keywords = ["Technology"]
categories = ["Technology"]
+++
近来要调研一些虚拟化的东西，主要包括轻量级虚拟化、边缘池算力等，记下一些关键的点以便后续整理。    

手头的两台GPU服务器，每台上面有7块GPU卡，需要将它们透传到虚拟机中以便调研相关系统的性能。    

### 环境
环境信息列举如下:    

```
CPU: model name      : Intel(R) Xeon(R) Gold 5118 CPU @ 2.30GHz
Memory: 
free -m
              total        used        free      shared  buff/cache   available
Mem:         385679       22356      284811        1042       78512      358158
Swap:             0           0           0
GPU: 
# lspci -nn | grep -i nvidia
3d:00.0 3D controller [0302]: NVIDIA Corporation GV100GL [Tesla V100 PCIe 32GB] [10de:xxxx] (rev a1)
3e:00.0 3D controller [0302]: NVIDIA Corporation GV100GL [Tesla V100 PCIe 32GB] [10de:xxxx] (rev a1)
40:00.0 3D controller [0302]: NVIDIA Corporation GV100GL [Tesla V100 PCIe 32GB] [10de:xxxx] (rev a1)
41:00.0 3D controller [0302]: NVIDIA Corporation GV100GL [Tesla V100 PCIe 32GB] [10de:xxxx] (rev a1)
b1:00.0 3D controller [0302]: NVIDIA Corporation GV100GL [Tesla V100 PCIe 32GB] [10de:xxxx] (rev a1)
b2:00.0 3D controller [0302]: NVIDIA Corporation GV100GL [Tesla V100 PCIe 32GB] [10de:xxxx] (rev a1)
b4:00.0 3D controller [0302]: NVIDIA Corporation GV100GL [Tesla V100 PCIe 32GB] [10de:xxxx] (rev a1)
b5:00.0 3D controller [0302]: NVIDIA Corporation GV100GL [Tesla V100 PCIe 32GB] [10de:xxxx] (rev a1)
```
操作系统：   

```
# cat /etc/redhat-release 
CentOS Linux release 7.6.1810 (Core) 
# /usr/libexec/qemu-kvm --version
QEMU emulator version 2.12.0 (qemu-kvm-ev-2.12.0-44.1.el7_8.1)
Copyright (c) 2003-2017 Fabrice Bellard and the QEMU Project developers
# uname -r
4.19.12-1.el7.elrepo.x86_64
```

### 系统配置
激活IOMMU， 通过编辑`/etc/default/grub`中的`GRUB_CMDLINE_LINUX`行，增加`intel_iommu=on`后，重新生成grub引导文件：    

```
# vim /etc/default/grub
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet intel_iommu=on rd.driver.pre=vfio-pci"
# grub-mkconfig -o /boot/grub2/grub.cfg
# reboot
重启后，检查：   
dmesg | grep -E "DMAR|IOMMU"

```
激活`vfio-pci`内核模块， 注意填入的数值是`lspci`取得的:    

```
# options vfio-pci ids=10de:xxxx
```
激活`vfio-pci`的自动加载:    

```
# echo 'vfio-pci' > /etc/modules-load.d/vfio-pci.conf
# reboot
# dmesg | grep -i vfio
```
qemu的更新步骤如下:    

```
# /usr/libexec/qemu-kvm -version
QEMU emulator version 1.5.3 (qemu-kvm-1.5.3-141.el7_4.6), Copyright (c) 2003-2008 Fabrice Bellard
# yum -y install centos-release-qemu-ev
# sed -i -e "s/enabled=1/enabled=0/g" /etc/yum.repos.d/CentOS-QEMU-EV.repo
# yum --enablerepo=centos-qemu-ev -y install qemu-kvm-ev
# systemctl restart libvirtd
# /usr/libexec/qemu-kvm -version
QEMU emulator version 2.12.0 (qemu-kvm-ev-2.12.0-44.1.el7_8.1)
```
现在在virt-manager中是可以指定下放GPU的。   

![/images/2021_06_05_08_17_16_785x569.jpg](/images/2021_06_05_08_17_16_785x569.jpg)

登录到虚拟机以后：   

```
root@localhost:~# lspci -nn | grep -i nvidia
00:0a.0 3D controller [0302]: NVIDIA Corporation GV100GL [Tesla V100 PCIe 32GB] [10de:xxxx] (rev a1)
```

接下来我会调研如何在lxd下及在k3s+kubevirt的场景下透传GPU.     
