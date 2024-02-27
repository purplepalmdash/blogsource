+++
title= "QuicklyDumpAMI"
date = "2024-02-27T08:50:42+08:00"
description = "QuicklyDumpAMI"
keywords = ["Technology"]
categories = ["Technology"]
+++
### win10 Quickstart
Quickly partition:    

![/images/2024_02_27_08_52_43_703x505.jpg](/images/2024_02_27_08_52_43_703x505.jpg)

Like following layout:    

![/images/2024_02_27_08_53_19_777x119.jpg](/images/2024_02_27_08_53_19_777x119.jpg)

Insert Cloudfirmware flash disk , copy the files:    

![/images/2024_02_27_09_05_38_503x197.jpg](/images/2024_02_27_09_05_38_503x197.jpg)

![/images/2024_02_27_09_05_50_543x494.jpg](/images/2024_02_27_09_05_50_543x494.jpg)

Download the `AMI FIRMWARE UPDATE(AFU)` FROM:    

`https://www.ami.com/bios-uefi-utilities/`    

![/images/2024_02_27_09_59_36_289x147.jpg](/images/2024_02_27_09_59_36_289x147.jpg)

![/images/2024_02_27_09_59_46_885x672.jpg](/images/2024_02_27_09_59_46_885x672.jpg)

Click `save` to save the current rom:    

![/images/2024_02_27_10_00_16_804x566.jpg](/images/2024_02_27_10_00_16_804x566.jpg)

View the result:    

![/images/2024_02_27_10_00_44_937x134.jpg](/images/2024_02_27_10_00_44_937x134.jpg)

### extract files
Using `UBU_v1.79.17`:    

![/images/2024_02_27_10_25_36_553x214.jpg](/images/2024_02_27_10_25_36_553x214.jpg)

put `afuwin.rom` into this folder:   

![/images/2024_02_27_10_26_20_704x397.jpg](/images/2024_02_27_10_26_20_704x397.jpg)

Wait util scanning finished:   

![/images/2024_02_27_10_28_12_671x454.jpg](/images/2024_02_27_10_28_12_671x454.jpg)

Enter next step and Press `2`:    

![/images/2024_02_27_10_29_08_589x619.jpg](/images/2024_02_27_10_29_08_589x619.jpg)

Press `S` for share this:    

![/images/2024_02_27_10_30_35_585x377.jpg](/images/2024_02_27_10_30_35_585x377.jpg)

![/images/2024_02_27_10_30_51_538x511.jpg](/images/2024_02_27_10_30_51_538x511.jpg)

Get the `IntelGopDriver.efi`:    

![/images/2024_02_27_10_43_33_915x158.jpg](/images/2024_02_27_10_43_33_915x158.jpg)

Extract using `edk2-BaseTools`:     

```
EfiRom.exe -f 0x8086 -i 0x46d1 -e IntelGopDriver.efi
```

![/images/2024_02_27_10_50_30_1006x399.jpg](/images/2024_02_27_10_50_30_1006x399.jpg)


![/images/2024_02_27_10_29_46_523x387.jpg](/images/2024_02_27_10_29_46_523x387.jpg)

### pve configuration
```
cd /etc/apt
cp sources.list sources.list.back
sed -i 's|^deb http://ftp.debian.org|deb https://mirrors.ustc.edu.cn|g' /etc/apt/sources.list
sed -i 's|^deb http://security.debian.org|deb https://mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
sed -i 's|^deb https://enterprise.proxmox.com|deb https://mirrors.ustc.edu.cn|g' /etc/apt/sources.list.d/ceph.list
sed -i 's|enterprise|no-subscription|g' /etc/apt/sources.list.d/ceph.list
source /etc/os-release
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve $VERSION_CODENAME pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
cp /usr/share/perl5/PVE/APLInfo.pm /usr/share/perl5/PVE/APLInfo.pm_back
sed -i 's|http://download.proxmox.com|https://mirrors.ustc.edu.cn/proxmox|g' /usr/share/perl5/PVE/APLInfo.pm
mv /etc/apt/sources.list.d/pve-enterprise.list /root
```
添加概要信息:    

```
wget -q -O /root/pve_source.tar.gz 'https://bbs.x86pi.cn/file/topic/2023-11-28/file/01ac88d7d2b840cb88c15cb5e19d4305b2.gz' && tar zxvf /root/pve_source.tar.gz && /root/./pve_source
```
Install vim:    

```
apt install -y vim 
```
edit the grub and blacklists:   

```
# vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on initcall_blacklist=sysfb_init"
# update-grub2
# vim /etc/modprobe.d/pve-blacklist.conf
blacklist nvidiafb
blacklist amdgpu
blacklist i915
blacklist snd_hda_intel
options vfio_iommu_type1 allow_unsafe_interrupts=1
# update-initramfs -u -k all

```
![/images/2024_02_27_11_31_32_642x219.jpg](/images/2024_02_27_11_31_32_642x219.jpg)

Get the pciids:   

```
root@pve:~# lspci -D -nn | grep VGA
0000:00:02.0 VGA compatible controller [0300]: Intel Corporation Alder Lake-N [UHD Graphics] [8086:46d1]
root@pve:~# lspci -D -nn | grep Audio
0000:00:1f.3 Audio device [0403]: Intel Corporation Device [8086:54c8]
```
Prepare the rom files:     

```
root@pve:~/igd-main# cp gen12_gop.rom gen12_igd.rom /usr/share/kvm/
root@pve:~/igd-main# pwd
/root/igd-main
# cp IntelGopDriver.rom /usr/share/kvm/
```

### vm setup
Configuration:    

![/images/2024_02_27_11_52_11_718x277.jpg](/images/2024_02_27_11_52_11_718x277.jpg)

![/images/2024_02_27_11_53_17_715x327.jpg](/images/2024_02_27_11_53_17_715x327.jpg)

![/images/2024_02_27_11_53_52_632x308.jpg](/images/2024_02_27_11_53_52_632x308.jpg)

![/images/2024_02_27_11_54_34_720x153.jpg](/images/2024_02_27_11_54_34_720x153.jpg)

![/images/2024_02_27_11_55_09_727x215.jpg](/images/2024_02_27_11_55_09_727x215.jpg)

Hardware Adjust:    

![/images/2024_02_27_11_56_01_419x285.jpg](/images/2024_02_27_11_56_01_419x285.jpg)

Add pci/usb devices:    

![/images/2024_02_27_11_58_57_426x119.jpg](/images/2024_02_27_11_58_57_426x119.jpg)

qemu commands:    

```
root       14422       1 99 12:04 ?        00:45:17 /usr/bin/kvm -id 100 -name win10,debug-threads=on -no-shutdown -chardev socket,id=qmp,path=/var/run/qemu-server/100.qmp,server=on,wait=off -mon chardev=qmp,mode=control -chardev socket,id=qmp-event,path=/var/run/qmeventd.sock,reconnect=5 -mon chardev=qmp-event,mode=control -pidfile /var/run/qemu-server/100.pid -daemonize -smbios type=1,uuid=b886e2ce-0caf-42a3-84db-d3568b369292 -drive if=pflash,unit=0,format=raw,readonly=on,file=/usr/share/pve-edk2-firmware//OVMF_CODE_4M.fd -drive if=pflash,unit=1,id=drive-efidisk0,format=qcow2,file=/var/lib/vz/images/100/vm-100-disk-0.qcow2 -smp 4,sockets=1,cores=4,maxcpus=4 -nodefaults -boot menu=on,strict=on,reboot-timeout=1000,splash=/usr/share/qemu-server/bootsplash.jpg -vga none -nographic -cpu host,hv_ipi,hv_relaxed,hv_reset,hv_runtime,hv_spinlocks=0x1fff,hv_stimer,hv_synic,hv_time,hv_vapic,hv_vpindex,+kvm_pv_eoi,+kvm_pv_unhalt -m 8192 -object iothread,id=iothread-virtioscsi0 -device pci-bridge,id=pci.1,chassis_nr=1,bus=pci.0,addr=0x1e -device pci-bridge,id=pci.2,chassis_nr=2,bus=pci.1,addr=0x1e -device pci-bridge,id=pci.3,chassis_nr=3,bus=pci.0,addr=0x5 -device vmgenid,guid=e489d935-fb57-482a-9c07-0a14cb77f1e7 -device piix3-usb-uhci,id=uhci,bus=pci.0,addr=0x1.0x2 -device qemu-xhci,p2=15,p3=15,id=xhci,bus=pci.1,addr=0x1b -device usb-tablet,id=tablet,bus=uhci.0,port=1 -device vfio-pci,host=0000:00:02.0,id=hostpci0,bus=pci.0,addr=0x2,romfile=/usr/share/kvm/gen12_igd.rom -device vfio-pci,host=0000:00:1f.3,id=hostpci1,bus=pci.0,addr=0x11,romfile=/usr/share/kvm/IntelGopDriver.rom -device usb-host,bus=xhci.0,port=1,hostbus=1,hostport=1,id=usb0 -device usb-host,bus=xhci.0,port=2,hostbus=1,hostport=4,id=usb1 -device virtio-balloon-pci,id=balloon0,bus=pci.0,addr=0x3,free-page-reporting=on -iscsi initiator-name=iqn.1993-08.org.debian:01:7462d45a788 -drive file=/var/lib/vz/template/iso/virtio-win-0.1.208.iso,if=none,id=drive-ide0,media=cdrom,aio=io_uring -device ide-cd,bus=ide.0,unit=0,drive=drive-ide0,id=ide0,bootindex=102 -drive file=/var/lib/vz/template/iso/Win10_22H2_Chinese_Simplified_x64.iso,if=none,id=drive-ide2,media=cdrom,aio=io_uring -device ide-cd,bus=ide.1,unit=0,drive=drive-ide2,id=ide2,bootindex=101 -device virtio-scsi-pci,id=virtioscsi0,bus=pci.3,addr=0x1,iothread=iothread-virtioscsi0 -drive file=/var/lib/vz/images/100/vm-100-disk-1.qcow2,if=none,id=drive-scsi0,format=qcow2,cache=none,aio=io_uring,detect-zeroes=on -device scsi-hd,bus=virtioscsi0.0,channel=0,scsi-id=0,lun=0,drive=drive-scsi0,id=scsi0,bootindex=100 -netdev type=tap,id=net0,ifname=tap100i0,script=/var/lib/qemu-server/pve-bridge,downscript=/var/lib/qemu-server/pve-bridgedown,vhost=on -device virtio-net-pci,mac=BC:24:11:33:3E:BB,netdev=net0,bus=pci.0,addr=0x12,id=net0,rx_queue_size=1024,tx_queue_size=256 -rtc driftfix=slew,base=localtime -machine hpet=off,type=pc-i440fx-8.1+pve0 -global kvm-pit.lost_tick_policy=discard -set device.hostpci0.addr=02.0 -set device.hostpci0.x-igd-gms=0x2 -set device.hostpci0.x-igd-opregion=on

```
