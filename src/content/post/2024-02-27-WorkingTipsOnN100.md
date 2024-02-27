+++
title= "WorkingTipsOnN100"
date = "2024-02-27T20:19:41+08:00"
description = "WorkingTipsOnN100"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. Extract ROM
Using AFU(AMI FIRMWARE UPDATE) for extract the uefi rom file:   

![/images/2024_02_27_09_59_36_289x147.jpg](/images/2024_02_27_09_59_36_289x147.jpg)

![/images/2024_02_27_09_59_46_885x672.jpg](/images/2024_02_27_09_59_46_885x672.jpg)

Click `save` to save the current rom:    

![/images/2024_02_27_10_00_16_804x566.jpg](/images/2024_02_27_10_00_16_804x566.jpg)

View the result:    

![/images/2024_02_27_10_00_44_937x134.jpg](/images/2024_02_27_10_00_44_937x134.jpg)

Using `UBU_v1.79.17` for extracting `IntelGopDriver.efi`:    

![/images/2024_02_27_10_25_36_553x214.jpg](/images/2024_02_27_10_25_36_553x214.jpg)

put `afuwin.rom`(extracted via AFU) into the same folder:   

![/images/2024_02_27_10_26_20_704x397.jpg](/images/2024_02_27_10_26_20_704x397.jpg)

Wait util scanning finished:   

![/images/2024_02_27_10_28_12_671x454.jpg](/images/2024_02_27_10_28_12_671x454.jpg)

Enter next step and Press `2`:    

![/images/2024_02_27_10_29_08_589x619.jpg](/images/2024_02_27_10_29_08_589x619.jpg)

Press `S` for share this:    

![/images/2024_02_27_10_30_35_585x377.jpg](/images/2024_02_27_10_30_35_585x377.jpg)

![/images/2024_02_27_10_30_51_538x511.jpg](/images/2024_02_27_10_30_51_538x511.jpg)

Now you get the `IntelGopDriver.efi` file for building OVMF:    

![/images/2024_02_27_10_43_33_915x158.jpg](/images/2024_02_27_10_43_33_915x158.jpg)

### 2. Building OVMF
Building the OVMF via following commands:    

```
git clone git@github.com:cmd2001/build-edk2-gvtd.git
cd build-edk2-gvtd
sh ./init_edk2.sh
mkdir -p gop
cp <intel gop driver efi> gop/IntelGopDriver.efi
sudo bash ./build_ovmf.sh
sudo bash ./build_oprom.sh
```
Your generated OVMF and rom files should be like following:    

```
$ ls product/ -l -h
total 7.4M
-rw-r--r-- 1 root root 186K Feb 27 19:29 B660_GOP.rom
-rw-r--r-- 1 root root 213K Feb 27 19:29 B660.rom
-rw-r--r-- 1 root root 3.5M Feb 27 19:20 OVMF_CODE_4M.fd
-rw-r--r-- 1 root root 3.5M Feb 27 19:22 OVMF_CODE_4M.secboot.fd
```
Copy the files to N100 machine, rename `OVMF_CODE_4M.fd` to `own_OVMF_CODE_4M.fd`, `OVMF_CODE_4M.secboot.fd` to `own_OVMF_CODE_4M.secboot.fd`, save them under the folder `/usr/share/OVMF/`:    

```
root@n100:~# ls /usr/share/OVMF/own_OVMF_CODE_4M.*
/usr/share/OVMF/own_OVMF_CODE_4M.fd  /usr/share/OVMF/own_OVMF_CODE_4M.secboot.fd
root@n100:~# ls /usr/share/OVMF/B660*
/usr/share/OVMF/B660_GOP.rom  /usr/share/OVMF/B660.rom
```
### 3. n100 OS setup
Hardware/Software details:   

```
# uname -r
6.1.30-victory
# cat /etc/issue
Ubuntu 18.04.6 LTS \n \l
# lscpu | grep -i model
Model:               190
Model name:          Intel(R) N100
# free -m
              total        used        free      shared  buff/cache   available
Mem:          31876        9263       16566           1        6046       22217
Swap:          2047           0        2047
```
Grub default configration:    

```
# vim /etc/default/grub
...
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash processor.max_cstate=0 intel_idle.max_cstate=0 intel_iommu=on iommu=pt split_lock_detect=off intel_iommu=pt kvm.ignore_msrs=1 video=efifb:off,vesafb:off initcall_backlist=sysfb_init pcie_acs_override=downstream,multifunction"
...
# update-grub2
# vim /etc/modprobe.d/blacklist.conf
...
...
blacklist i915
blacklist snd_hda_intel
blacklist snd_hda_codec_hdmi
options vfio_iommu_type1 allow_unsafe_interrupts=1
# vim /etc/initramfs-tools/module
...
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
# update-initramfs -u -k all
```

### 4. Qemu
Build qemu via:    

```
git clone https://git.qemu.org/git/qemu.git
cd qemu
git checkout v8.0.1
git submodule init
git submodule update --recursive
mkdir -p build
cd build
mkdir -p /opt/local1
../configure --prefix=/opt/local --enable-kvm --disable-xen --enable-libusb --enable-debug-info --enable-debug --enable-sdl --enable-vhost-net --enable-spice --disable-debug-tcg --enable-opengl --enable-gtk --target-list=x86_64-softmmu --audio-drv-list=alsa
make -j4 && make install
```
Check the qemu version:    

```
# /opt/local1/bin/qemu-system-x86_64 --version
QEMU emulator version 8.0.1 (v8.0.1)
Copyright (c) 2003-2022 Fabrice Bellard and the QEMU Project developers
```
### 5. vm setup
Prepare the iso and qcow2 files:    

```
# pwd
/root
# ls *.iso
virtio-win-0.1.208.iso  Win10_22H2_Chinese_Simplified_x64.iso
# qemu-img create -f qcow2 win10_base.qcow2 100G
```
Prepare the vfio pci devices:    

```
# vfio-pci.sh -h 00:02.0 && vfio-pci.sh -h 00:14.0
# lspci -s 00:02.0 -vv | grep -i driver
	Kernel driver in use: vfio-pci
# lspci -s 00:14.0 -vv | grep -i driver
	Kernel driver in use: vfio-pci
```
Create the machine via:    

```
sudo /opt/local1/bin/qemu-system-x86_64 -no-user-config -nodefaults -m 8192M,slots=4,maxmem=16G -enable-kvm \
-machine pc,accel=kvm,kernel_irqchip=on,mem-merge=off \
-drive file=/usr/share/OVMF/own_OVMF_CODE_4M.fd,format=raw,if=pflash \
-cpu host,hv_relaxed,hv-vapic,hv-spinlocks=4096,hv-time,hv-runtime,hv-synic,hv-stimer,hv_vpindex,hv-tlbflush,hv-ipi \
-smp cores=1,threads=2,sockets=2 \
-device vfio-pci,host=00:02.0,id=vga0,bus=pci.0,addr=0x2,x-igd-gms=6,x-igd-opregion=on,romfile=/usr/share/OVMF/B660_GOP.rom \
-device vfio-pci,host=0000:00:14.0,id=hostpci1,bus=pci.0,addr=0x11 \
-drive file=/root/win10_base.qcow2,format=qcow2,cache=none,if=none,id=drv0 -device virtio-blk,drive=drv0,id=vdisk0 \
-drive file=/root/Win10_22H2_Chinese_Simplified_x64.iso,media=cdrom -drive file=/root/virtio-win-0.1.208.iso,media=cdrom  \
-vga none -nographic -netdev user,hostfwd=tcp::2288-:22,hostfwd=tcp::13389-:3389,id=net0 -device virtio-net-pci,netdev=net0,mac=11:22:33:44:55:66 \
-monitor telnet:localhost:2222,server,nowait 
```
You will get the installation window directly on Monitor, simply install the system, when system ready, get the latest i915 driver from intel website(`gfx_win_101.5333`).     

### 6. Result
Device Driver:    

![/images/2024_02_27_20_39_36_614x529.jpg](/images/2024_02_27_20_39_36_614x529.jpg)

Task manager details:    

![/images/2024_02_27_20_40_07_645x516.jpg](/images/2024_02_27_20_40_07_645x516.jpg)

WebGL fish:    

![/images/2024_02_27_20_38_53_567x400.jpg](/images/2024_02_27_20_38_53_567x400.jpg)

Video decoding(1080p 60fps):    

![/images/2024_02_27_20_45_05_1440x703.jpg](/images/2024_02_27_20_45_05_1440x703.jpg)


