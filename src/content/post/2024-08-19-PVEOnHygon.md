+++
title= "PVEOnHygon"
date = "2024-08-19T08:23:51+08:00"
description = "PVEOnHygon"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Host configuration
Hardware Info:    

```
# cat /proc/cpuinfo | grep -i "model name"
model name	: Hygon C86 3350  8-core Processor
```
Os Info:    

```
root@pve:~# cat /etc/debian_version 
12.4
root@pve:~# uname -a
Linux pve 6.5.11-8-pve #1 SMP PREEMPT_DYNAMIC PMX 6.5.11-8 (2024-01-30T12:27Z) x86_64 GNU/Linux
```
Modifications :      

```
# cat /etc/default/grub | grep amd_iommu
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt amd_iommu=on initcall_blacklist=sysfb_init pcie_acs_override=downstream,multifunction"
# cat /etc/modules
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
# cat /etc/modprobe.d/vfio.conf
options vfio-pci ids=1002:6611,1002:aab0
options vfio-pci disable_idle_d3=1
# update-initramfs -u -k all && update-grub && reboot
```
### virtualization
Win10 configuration:    

![/images/20240819_082807_x.jpg](/images/20240819_082807_x.jpg)

you can get win10 working properly directly, but for win7, it could only boot into system, after driver installation, it will remains black and could not startup.    

### amd radeon 550
Added more vfio items:    

```
# cat /etc/modprobe.d/vfio.conf 
options vfio-pci ids=1002:6611,1002:aab0,1002:699f,1002:aae0
options vfio-pci disable_idle_d3=1
# update-initramfs -u -k all && update-grub && reboot
```
The same result as radeon 520.     


Changed back to an old driver, OK.    

```
-rw-rw-r-- 1 dash dash  478517432  8月 19 09:27 non-whql-win7-64bit-radeon-software-crimson-relive-17.5.1-may4.exe
```

### Re-installation

![/images/20240819_095520_x.jpg](/images/20240819_095520_x.jpg)

commands:    

```
root        1308       1 80 15:40 ?        00:01:22 /usr/bin/kvm -id 107 -name 107107,debug-threads=on -no-shutdown -chardev socket,id=qmp,path=/var/run/qemu-server/107.qmp,server=on,wait=off -mon chardev=qmp,mode=control -chardev socket,id=qmp-event,path=/var/run/qmeventd.sock,reconnect=5 -mon chardev=qmp-event,mode=control -pidfile /var/run/qemu-server/107.pid -daemonize -smbios type=1,uuid=67b73609-9afe-4dd0-93b8-df42b3e114b5 -smp 2,sockets=1,cores=2,maxcpus=2 -nodefaults -boot menu=on,strict=on,reboot-timeout=1000,splash=/usr/share/qemu-server/bootsplash.jpg -vga none -nographic -cpu qemu64,+aes,enforce,hv_ipi,hv_relaxed,hv_reset,hv_runtime,hv_spinlocks=0x1fff,hv_stimer,hv_synic,hv_time,hv_vapic,hv_vendor_id=proxmox,hv_vpindex,kvm=off,+kvm_pv_eoi,+kvm_pv_unhalt,+pni,+popcnt,+sse4.1,+sse4.2,+ssse3 -m 3072 -readconfig /usr/share/qemu-server/pve-q35-4.0.cfg -device vmgenid,guid=cb144ce1-60a1-4af3-8842-6b47fa91f4df -device usb-tablet,id=tablet,bus=ehci.0,port=1 -device vfio-pci,host=0000:06:00.0,id=hostpci0,bus=pcie.0,addr=0x10,x-vga=on -device usb-host,vendorid=0x30fa,productid=0x0300,id=usb0 -device usb-host,hostbus=1,hostport=1.1,id=usb1 -device virtio-balloon-pci,id=balloon0,bus=pci.0,addr=0x3,free-page-reporting=on -iscsi initiator-name=iqn.1993-08.org.debian:01:ba1fd0778bd -drive file=/var/lib/vz/template/iso/Win7_hygon.iso,if=none,id=drive-ide2,media=cdrom,aio=io_uring -device ide-cd,bus=ide.1,unit=0,drive=drive-ide2,id=ide2,bootindex=101 -device ahci,id=ahci0,multifunction=on,bus=pci.0,addr=0x7 -drive file=/dev/pve/vm-107-disk-0,if=none,id=drive-sata0,format=raw,cache=none,aio=io_uring,detect-zeroes=on -device ide-hd,bus=ahci0.0,drive=drive-sata0,id=sata0,bootindex=100 -netdev type=tap,id=net0,ifname=tap107i0,script=/var/lib/qemu-server/pve-bridge,downscript=/var/lib/qemu-server/pve-bridgedown -device e1000,mac=BC:24:11:9A:C1:70,netdev=net0,bus=pci.0,addr=0x12,id=net0,bootindex=102 -rtc driftfix=slew,base=localtime -machine hpet=off,smm=off,type=pc-q35-8.1+pve0 -global kvm-pit.lost_tick_policy=discard
```
