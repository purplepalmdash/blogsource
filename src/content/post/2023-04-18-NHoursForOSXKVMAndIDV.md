+++
title= "NHoursForOSXKVMAndIDV"
date = "2023-04-18T08:33:44+08:00"
description = "NHoursForOSXKVMAndIDV"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 系统硬件
主要信息:    

```
华擎Z370M主板
Intel i7-9700K
32G 内存
200G SSD存储
```
![/images/2023_04_18_09_51_43_990x574.jpg](/images/2023_04_18_09_51_43_990x574.jpg)

附加显卡信息:    

```
$ sudo lspci | grep -i vga
00:02.0 VGA compatible controller: Intel Corporation CoffeeLake-S GT2 [UHD Graphics 630]
01:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Baffin [Radeon RX 550 640SP / RX 560/560X] (rev ff)
03:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Turks [Radeon HD 7600 Series]
```

### BIOS配置及选项详解
BIOS `高级->CPU配置`中，打开`Intel虚拟化技术`，关闭`Software Guard Extensions (SGX)`:    

![/images/2023_04_18_09_54_10_1232x252.jpg](/images/2023_04_18_09_54_10_1232x252.jpg)

BIOS `高级->芯片组配置`中，打开`VT-d`功能，关闭`Above 4G Decoding`， 主图形适配器选择为`板载`:    

![/images/2023_04_18_09_55_18_1198x437.jpg](/images/2023_04_18_09_55_18_1198x437.jpg)

BIOS `高级->芯片组配置`中，设置显卡的`共享内存`为1G, 打开IGPU多监视器选项，使得在外部有显卡的情况下集显保持开启:    

![/images/2023_04_18_09_58_09_1130x153.jpg](/images/2023_04_18_09_58_09_1130x153.jpg)

(可选)关闭可信赖计算:    

![/images/2023_04_18_09_59_00_1115x241.jpg](/images/2023_04_18_09_59_00_1115x241.jpg)

引导->CSM(兼容性支持模块)中，将`启动视频OpROM策略`设置为`仅传统`:    

![/images/2023_04_18_10_03_57_1201x270.jpg](/images/2023_04_18_10_03_57_1201x270.jpg)

调整启动顺序为从U盘启动，进入到系统的安装过程。

### Host系统安装
将网上下载的Ubuntu22.04安装盘写入U盘:    

```
$ sudo dd if=ubuntu-22.04.2-live-server-amd64.iso of=/dev/sdb bs=1M && sudo sync
```
选择UEFI分区的USB设备进入到安装过程:   

![/images/2023_04_18_10_24_27_844x331.jpg](/images/2023_04_18_10_24_27_844x331.jpg)

Grub栏中，选择`Try or Install Ubuntu Server`:    

![/images/2023_04_18_10_05_47_1120x296.jpg](/images/2023_04_18_10_05_47_1120x296.jpg)

选择语言为`English`，自动配置网络， 忽略proxy address配置，mirror address保持默认（也可以改变为网速较好的源) :   

![/images/2023_04_18_10_10_20_828x234.jpg](/images/2023_04_18_10_10_20_828x234.jpg)

(非必选)分区这里选择基本(避免LVM可能给新学者带来的困扰)：   

![/images/2023_04_18_10_11_08_1039x286.jpg](/images/2023_04_18_10_11_08_1039x286.jpg)

检查下存储layout是否正常，无问题则下一步:    

![/images/2023_04_18_10_11_22_1209x570.jpg](/images/2023_04_18_10_11_22_1209x570.jpg)

![/images/2023_04_18_10_12_05_876x301.jpg](/images/2023_04_18_10_12_05_876x301.jpg)
配置用户名/密码:    

![/images/2023_04_18_10_25_59_1091x440.jpg](/images/2023_04_18_10_25_59_1091x440.jpg)

忽略`Ubuntu Pro`选项后继续，选择`Install OpenSSH server`:    

![/images/2023_04_18_10_26_44_919x365.jpg](/images/2023_04_18_10_26_44_919x365.jpg)

在子功能选单中，什么都不选，进入到安装过程，直到所有安装都完成。Ubuntu live server在安装时最好保持联网状态，否则可能出现部分包安装不成功导致整个系统安装失败的情况。

看到这个界面则显示成功:   

![/images/2023_04_18_10_31_49_1509x1014.jpg](/images/2023_04_18_10_31_49_1509x1014.jpg)

安装后登陆界面:  

![/images/2023_04_18_10_33_43_860x394.jpg](/images/2023_04_18_10_33_43_860x394.jpg)
    
### Host系统安装问题解决
在安装过pve的磁盘上,或者以前用LVM安装过系统的硬盘上，全新安装Ubuntu时，会出现以下的错误：   

![/images/2023_04_18_10_21_19_1400x506.jpg](/images/2023_04_18_10_21_19_1400x506.jpg)
解决方法为:   

```
vgremove xxxxxx(这里填你的vg的名字，通过vgs查看)
pvremove /dev/xxxx(这里填你的pvs所在的实际物理设备，通过pvs查看)
```
完全删除掉已有的逻辑卷分区后，重新进入到安装过程后方可成功。

### 系统配置
确保系统更新到最新状态：    

```
sudo apt update
sudo apt upgrade -y
```
选择KDE作为默认的桌面工作环境, 并安装其他的一些包，选择KDE的原因在于它使用sddm+Xorg, 后续用libvirt钩子用于gvt-d透传集成显卡到虚机的时候，可以卸载得比较完整。默认的Ubuntu-desktop有时会出现在主机/虚机之间切换时失败的情况，推测与gdm3的工作方式有关：    

```
sudo apt install -y kubuntu-desktop virt-manager build-essential
```
配置vfio相关选项:    

```
# sudo vim /etc/default/grub
......
GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on intel_iommu=pt kvm.ignore_msrs=1"
......
# sudo update-grub2
# sudo vim /etc/initramfs-tools/modules
......
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
使用`lspci`查看两块AMD独显的VGA及Audio配置信息，记录下相应字段:    

![/images/2023_04_18_11_08_58_945x326.jpg](/images/2023_04_18_11_08_58_945x326.jpg)

```
# echo "options vfio-pci ids=1002:67ff,1002:675b,1002:aae0,1002:aa90" >> /etc/modprobe.d/vfio.conf
# sudo vim /etc/modprobe.d/blacklist.conf
......
blacklist amdgpu
# update-initramfs -u -k all && reboot
```
配置KDE的屏幕锁屏选项:    

![/images/2023_04_18_11_34_32_930x490.jpg](/images/2023_04_18_11_34_32_930x490.jpg)

关闭Energy Saving：   

![/images/2023_04_18_11_35_14_832x441.jpg](/images/2023_04_18_11_35_14_832x441.jpg)

Login Screen(SDDM)选择 Bahavior:    

![/images/2023_04_18_11_36_33_951x832.jpg](/images/2023_04_18_11_36_33_951x832.jpg)
使能登出后自动登入:   

![/images/2023_04_19_14_41_02_962x411.jpg](/images/2023_04_19_14_41_02_962x411.jpg)


之后选择子选项:    

![/images/2023_04_18_11_37_28_1057x607.jpg](/images/2023_04_18_11_37_28_1057x607.jpg)

KDE桌面下其实是存在大量的可优化空间的，后面可以深入探讨。  

### OSX-KVM落地
主要参考了: `https://github.com/kholia/OSX-KVM`:   

按仓库要求，安装相应包, 准备相关文件 :    

```
sudo apt-get install qemu uml-utilities virt-manager git \
    wget libguestfs-tools p7zip-full make dmg2img -y
sudo reboot
sudo usermod -aG kvm $(whoami)
sudo usermod -aG libvirt $(whoami)
sudo usermod -aG input $(whoami)
git clone --depth 1 --recursive https://github.com/kholia/OSX-KVM.git
```
下载Ventura (13):    

```
$ cd OSX-KVM
$ ./fetch-macOS-v2.py 
1. High Sierra (10.13)
2. Mojave (10.14)
3. Catalina (10.15)
4. Big Sur (11.7) - RECOMMENDED
5. Monterey (12.6)
6. Ventura (13)

Choose a product to download (1-6): 6
Ventura (13)
Downloading 032-66586...
Saving http://oscdn.apple.com/content/downloads/54/39/032-66586/dtdkth7btmat4w68aohv87st5j33d2puna/RecoveryImage/BaseSystem.dmg to BaseSystem.dmg...
Note: The total download size is 675.59 MB
$ dmg2img -i BaseSystem.dmg BaseSystem.img
$ qemu-img create -f qcow2 mac_hdd_ng.img 64G
$ ./OpenCore-Boot.sh 
```
进入到安装界面:    

![/images/2023_04_18_12_10_54_517x492.jpg](/images/2023_04_18_12_10_54_517x492.jpg)

选择Disk Utitity:    

![/images/2023_04_18_12_11_45_1088x442.jpg](/images/2023_04_18_12_11_45_1088x442.jpg)

而后选择`Reinstall`开始安装, 注意选择格式化后的macOS 64GB的盘:      

![/images/2023_04_18_12_13_57_583x567.jpg](/images/2023_04_18_12_13_57_583x567.jpg)


安装配置阶段:   

![/images/2023_04_18_13_10_06_801x599.jpg](/images/2023_04_18_13_10_06_801x599.jpg)

选择not now:   

![/images/2023_04_18_13_10_35_797x602.jpg](/images/2023_04_18_13_10_35_797x602.jpg)
选择 Set Up Later:   

![/images/2023_04_18_13_11_00_809x605.jpg](/images/2023_04_18_13_11_00_809x605.jpg)

创建用户:   

![/images/2023_04_18_13_11_35_711x403.jpg](/images/2023_04_18_13_11_35_711x403.jpg)

装完后:   

![/images/2023_04_18_13_15_45_290x510.jpg](/images/2023_04_18_13_15_45_290x510.jpg)

开启ssh和vnc:   

![/images/2023_04_18_14_09_08_710x390.jpg](/images/2023_04_18_14_09_08_710x390.jpg)

在命令行下，拷贝OpenBoot的EFI分区到硬盘下:    

```
test@tests-iMac-Pro ~ % sudo diskutil list
.......这里需要识别出哪个分区是ISO的，哪个是硬盘上的
test@tests-iMac-Pro ~ % sudo diskutil mount disk0s1
Volume EFI on disk0s1 mounted
test@tests-iMac-Pro ~ % sudo diskutil mount disk1s1
Volume EFI on disk1s1 mounted
test@tests-iMac-Pro ~ % mount
/dev/disk2s5s1 on / (apfs, sealed, local, read-only, journaled)
devfs on /dev (devfs, local, nobrowse)
/dev/disk2s4 on /System/Volumes/VM (apfs, local, noexec, journaled, noatime, nobrowse)
/dev/disk2s2 on /System/Volumes/Preboot (apfs, local, journaled, nobrowse)
/dev/disk2s6 on /System/Volumes/Update (apfs, local, journaled, nobrowse)
/dev/disk2s1 on /System/Volumes/Data (apfs, local, journaled, nobrowse)
map auto_home on /System/Volumes/Data/home (autofs, automounted, nobrowse)
/dev/disk0s1 on /Volumes/EFI (msdos, asynchronous, local, noowners)
/dev/disk1s1 on /Volumes/EFI 1 (msdos, asynchronous, local, noowners)
test@tests-iMac-Pro ~ % cp -r /Volumes/EFI/EFI /Volumes/EFI\ 1 
test@tests-iMac-Pro ~ % sudo sync     
test@tests-iMac-Pro ~ % sudo shutdown -h now
```
重新启动后，更改Boot的等待时延(挂在EFI分区后):    

```
 % cat EFI/OC/config.plist| grep '<key>Timeout' -A2
			<key>Timeout</key>
			<integer>5</integer>
		</dict>
```
### OSX-KVM with rx550(Ventura)
直接使用脚本测试，得到的结果：   

```
cp boot-passthrough.sh testrx550.sh
lspci | grep 550
01:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Baffin [Radeon RX 550 640SP / RX 560/560X] (rev ff)
01:00.1 Audio device: Advanced Micro Devices, Inc. [AMD/ATI] Baffin HDMI/DP Audio [Radeon RX 550 640SP / RX 560/560X]
vim testrx550.sh
  -vnc 0.0.0.0:8 -k en-us
  -netdev user,hostfwd=tcp::2288-:22,hostfwd=tcp::15900-:5900,id=net0 -device vmxnet3,netdev=net0,id=net0,mac=52:54:00:c9:18:27
```
效果： 最初引导的时候会出现画面，且可以一直操作(OpenCore),但是进入到苹果后黑屏。ssh可以登陆到成功启动后的macOS,但是vnc登陆则一直显示黑屏。   
### OSX-KVM(Monterey)
重新安装一台Monterey虚机:    

![/images/2023_04_18_15_14_39_508x574.jpg](/images/2023_04_18_15_14_39_508x574.jpg)

![/images/2023_04_18_16_15_35_595x362.jpg](/images/2023_04_18_16_15_35_595x362.jpg)

ssh/vnc启用:   

![/images/2023_04_18_16_16_21_657x516.jpg](/images/2023_04_18_16_16_21_657x516.jpg)

Monterey会正常启动

### vendor-reset
AMD显卡需要引入这个模块以便在vfio的时候reset为可用状态:     

```
git clone https://github.com/gnif/vendor-reset.git
cd vendor-reset/
sudo apt install -y dkms
sudo dkms install .
sudo vim /etc/initramfs-tools/modules
vendor-reset
sudo update-initramfs -u -k all
sudo reboot
```
重启成功后:     

```
$ lsmod | grep vendor
vendor_reset          114688  0
```
使用下面的`vfio_bind.sh`用来初始化显卡, 注意`vendor_reset`需要一个bug-fix（`device_specific`）:    

```
#!/bin/bash
modprobe vfio-pci
for dev in "$@"; do
vendor=$(cat /sys/bus/pci/devices/$dev/vendor)
device=$(cat /sys/bus/pci/devices/$dev/device)
if [ -e /sys/bus/pci/devices/$dev/driver ]; then
echo $dev > /sys/bus/pci/devices/$dev/driver/unbind
fi
echo $vendor $device > /sys/bus/pci/drivers/vfio-pci/new_id
echo 'device_specific' > /sys/bus/pci/devices/$dev/reset_method
done

```
### vbflash
使用工具dump出显卡的vbflash:    

![/images/2023_04_18_16_52_25_1024x467.jpg](/images/2023_04_18_16_52_25_1024x467.jpg)

7600看起来似乎无法驱动。临时换成了5700xt显卡用来测试。   
5700xt需要添加:     

```
test@tests-iMac-Pro EFI % cat EFI/OC/config.plist| grep boot-args -A3
				<key>boot-args</key>
				<string>-v keepsyms=1 tlbto_us=0 vti=9 agdpmod=pikera</string>
```

### 网络配置（桥接)
桥接是为了让虚拟机获得同网段地址:   

```
# sudo apt install bridge-utils -y
# sudo vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on intel_iommu=pt kvm.ignore_msrs=1 net.ifnames=0 biosdevname=0"
# sudo update-grub2
# vim /etc/netplan/00-installer-config.yaml 
# This is the network config written by 'subiquity'
network:
  ethernets:
    eth0:
      dhcp4: false
  bridges:
    br0:
      interfaces: [eth0]
      dhcp4: true
  version: 2
# reboot
```

### libvirtd集成
准备镜像:    

```
test@server:~/OSX-KVM-Monterey$ sudo cp mac_hdd_ng.img /var/lib/libvirt/images/ && sudo sync
test@server:~/OSX-KVM-Monterey$ sudo cp OVMF_VARS-1024x768.fd /var/lib/libvirt/images/
test@server:~/OSX-KVM-Monterey$ sudo cp OVMF_CODE.fd /var/lib/libvirt/images/
root@server:/var/lib/libvirt/images# qemu-img create -f qcow2 -b mac_hdd_ng.img -F qcow2 amd5700xt.qcow2
root@server:/var/lib/libvirt/images# qemu-img create -f qcow2 -b mac_hdd_ng.img -F qcow2 rx560.qcow2
root@server:/var/lib/libvirt/images# cp OVMF_VARS-1024x768.fd rx560_OVMF_VARS-1024x768.fd amd5700_OVMF_VARS-1024x768.fd
```
libvirtd配置文件(RX 5700 8GB及 RX 550 4GB):    

```
https://gist.githubusercontent.com/purplepalmdash/dfba3d32b72901f75d39a84288689692/raw/eb15ad7cd59f86566dd8fb5cfc6786639c0cb6a8/macOS5700.xml
https://gist.githubusercontent.com/purplepalmdash/dfba3d32b72901f75d39a84288689692/raw/eb15ad7cd59f86566dd8fb5cfc6786639c0cb6a8/macOSrx550.xml
```
### guest配置
阻止虚拟机进入到节能状态:    

![/images/2023_04_19_14_15_42_642x477.jpg](/images/2023_04_19_14_15_42_642x477.jpg)

禁止锁定屏幕:   

![/images/2023_04_19_14_17_15_622x403.jpg](/images/2023_04_19_14_17_15_622x403.jpg)

禁止自动更新:   

![/images/2023_04_19_14_18_38_638x318.jpg](/images/2023_04_19_14_18_38_638x318.jpg)

禁用屏幕保护:  

![/images/2023_04_19_14_19_26_627x551.jpg](/images/2023_04_19_14_19_26_627x551.jpg)


### win10核显透传
准备libvirtd钩子脚本:    

```
 cd /etc/libvirt/hooks/
 mv /home/test/hooks/* .
 ln -s /etc/libvirt/hooks/vfio-startup.sh /bin/
 ln -s /etc/libvirt/hooks/vfio-teardown.sh /bin/
```
如果要透传核显，则grub的参数需要做相应的修改:    

```
GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on intel_iommu=pt kvm.ignore_msrs=1 net.ifnames=0 biosdevname=0 video=efifb:off,vesafb:off"
```
因为以前已经做过相关的内容，这里就不再详细说明。   
### 已知/偶现问题
一次失败的核显透传后，重启失败, 更改grub默认选项后修复:    

![/images/2023_04_19_14_45_26_1018x372.jpg](/images/2023_04_19_14_45_26_1018x372.jpg)
Android studio无法运行?
