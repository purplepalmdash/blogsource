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


### Ubuntu20.04 changes
Switch back to Ubuntu20.04 then do following:    

```
$ sudo vim /etc/modprobe.d/vfio.conf
$ sudo systemctl set-default multi-user
Created symlink /etc/systemd/system/default.target → /lib/systemd/system/multi-user.target.
$ sudo reboot

Change the 

```
$ sudo vim /etc/initramfs-tools/scripts/init-top/vfio.sh
#!/bin/sh

PREREQ=""

prereqs()
{
   echo "$PREREQ"
}

case $1 in
prereqs)
   prereqs
   exit 0
   ;;
esac

for dev in 0000:b3:00.0 0000:b8:00.0 0000:bd:00.0 0000:c2:00.0
do 
 echo "vfio-pci" > /sys/bus/pci/devices/$dev/driver_override 
 echo "$dev" > /sys/bus/pci/drivers/vfio-pci/bind 
done

exit 0


$ sudo chmod +x /etc/initramfs-tools/scripts/init-top/vfio.sh
$ sudo vim /etc/initramfs-tools/modules
  And add the following lines:
  options kvm ignore_msrs=1
  vfio
  vfio_iommu_type1
  vfio_pci
  vfio_virqfd
```
Update the initramfs:    

```
# update-initramfs -c -k all
```

Examine the vfio driver usage(before vfio):    

```
lspci -v -s 0000:b8:00.0
0000:b8:00.0 VGA compatible controller: Intel Corporation Device 4907 (rev 01) (prog-if 00 [VGA controller])
	Subsystem: Hangzhou H3C Technologies Co., Ltd. Device 4000
	Flags: bus master, fast devsel, latency 0, IRQ 892, NUMA node 1
	Memory at e7000000 (64-bit, non-prefetchable) [size=16M]
	Memory at 386c00000000 (64-bit, prefetchable) [size=8G]
	Expansion ROM at e8000000 [disabled] [size=2M]
	Capabilities: [40] Vendor Specific Information: Len=0c <?>
	Capabilities: [70] Express Endpoint, MSI 00
	Capabilities: [ac] MSI: Enable+ Count=1/1 Maskable+ 64bit+
	Capabilities: [d0] Power Management version 3
	Capabilities: [100] Latency Tolerance Reporting
	Kernel driver in use: i915
	Kernel modules: i915

```
