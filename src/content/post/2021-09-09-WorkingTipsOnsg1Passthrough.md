+++
title= "WorkingTipsOnsg1Passthrough"
date = "2021-09-09T12:53:21+08:00"
description = "WorkingTipsOnsg1Passthrough"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 目的
透传sg1卡给虚拟机，同时支撑多个环境。


### 环境
物理机: `192.168.89.108`, 待开辟虚拟机`192.168.89.23~27`.     

```
[root@intel ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
[root@intel ~]# uname -a
Linux intel 4.14.105-6477750+ #1 SMP Mon May 17 10:31:49 CST 2021 x86_64 x86_64 x86_64 GNU/Linux
```
需透传的卡信息(`8086:4907`):    

```
[root@intel ~]# lspci -nn | grep -i vga
0000:05:00.0 VGA compatible controller [0300]: ASPEED Technology, Inc. ASPEED Graphics Family [1a03:2000] (rev 41)
0000:b3:00.0 VGA compatible controller [0300]: Intel Corporation Device [8086:4907] (rev 01)
0000:b8:00.0 VGA compatible controller [0300]: Intel Corporation Device [8086:4907] (rev 01)
0000:bd:00.0 VGA compatible controller [0300]: Intel Corporation Device [8086:4907] (rev 01)
0000:c2:00.0 VGA compatible controller [0300]: Intel Corporation Device [8086:4907] (rev 01)
```
### 开启vfio
修改内核参数并重启:    

```
# vim /etc/default/grub
...
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet i915.force_probe=* modprobe.blacklist=ast,snd_hda_intel i915.enable_guc=2 intel_iommu=on vfio-pci.ids=8086:4907"
...
修改modprobe.d规则:     

```
# vim /etc/modprobe.d/vfio.conf 
options vfio-pci ids=8086:4907
```
更新grub:    

```
# grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
```

### 配置网桥
安装:   

```
# yum install -y bridge-utils
```

