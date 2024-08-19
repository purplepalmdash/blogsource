+++
title= "KVMPassthroughOnHygon"
date = "2024-08-19T16:15:10+08:00"
description = "KVMPassthroughOnHygon"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Basic System Configuration Steps
Install some necessary packages:    

```
$ sudo vi /etc/apt/sources.list.d/ubuntu.sources
$ sudo apt update -y
$ sudo apt upgrade -y
$ sudo apt install -y virt-manager vim ovmf
$ qemu-system-x86_64 --version
QEMU emulator version 8.2.2 (Debian 1:8.2.2+ds-0ubuntu1)
Copyright (c) 2003-2023 Fabrice Bellard and the QEMU Project developers
$ sudo reboot
```
OS infos:    

```
idv@idv-P860:~$ uname -a
Linux idv-P860 6.8.0-40-generic #40-Ubuntu SMP PREEMPT_DYNAMIC Fri Jul  5 10:34:03 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
idv@idv-P860:~$ cat /etc/issue
Ubuntu 24.04 LTS \n \l
```
Unset some configuration under gnome:    

```
set Autologin
auto sleep, off
auto lock , off
```
### Virtualization Steps
Configure as following steps:    

```
# vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet iommu=pt amd_iommu=on initcall_blacklist=sysfb_init pcie_acs_override=downstream,multifunction"
# vim /etc/initramfs-tools/modules 
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
# vim /etc/modprobe.d/vfio.conf
options vfio-pci ids=1002:6611,1002:aab0
options vfio-pci disable_idle_d3=1
# update-initramfs -u -k all && update-grub2 && reboot
```

### VM Preparation(Win7-bios-q35)
Image creation:    

```
# qemu-img create -f qcow2 win7_bios_q35.qcow2 60G
Formatting 'win7_bios_q35.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off compression_type=zlib size=64424509440 lazy_refcounts=off refcount_bits=16
```

![/images/20240819_163149_x.jpg](/images/20240819_163149_x.jpg)

![./images/20240819_163206_x.jpg](./images/20240819_163206_x.jpg)

![./images/20240819_163236_x.jpg](./images/20240819_163236_x.jpg)

![./images/20240819_163253_x.jpg](./images/20240819_163253_x.jpg)

![./images/20240819_163327_x.jpg](./images/20240819_163327_x.jpg)

Install windows 7 as the default configuration.   


### VM Preparation(Win7-uefi-q35)
Create the disk via:   

```
# qemu-img create -f qcow2 win7_uefi_q35.qcow2 60G
Formatting 'win7_uefi_q35.qcow2', fmt=qcow2 cluster_size=65536 extended_l2=off compression_type=zlib size=64424509440 lazy_refcounts=off refcount_bits=16
```

![./images/20240819_163641_x.jpg](./images/20240819_163641_x.jpg)

![./images/20240819_163653_x.jpg](./images/20240819_163653_x.jpg)

Change the boot configuration:    

![/images/20240819_163741_x.jpg](/images/20240819_163741_x.jpg)

![/images/20240819_163839_x.jpg](/images/20240819_163839_x.jpg)

Using `Win7_Hygon.iso` we got abolve result.    


change to another iso,   `en_windows_7_ultimate_with_sp1_x64_dvd_u_677332.iso`:    

hang at following graphic:    

![/images/20240819_164144_x.jpg](/images/20240819_164144_x.jpg)

### libvirt hooks
After added vfio hooks, the command is listed as:    


```
/usr/bin/qemu-system-x86_64 -name guest=win7_bios_q35,debug-threads=on -S -object {"qom-type":"secret","id":"masterKey0","format":"raw","file":"/var/lib/libvirt/qemu/domain-2-win7_bios_q35/master-key.aes"} -machine pc-q35-8.2,usb=off,vmport=off,dump-guest-core=off,memory-backend=pc.ram,hpet=off,acpi=on -accel kvm -cpu host,migratable=on,hv-time=on,hv-relaxed=on,hv-vapic=on,hv-spinlocks=0x1fff -m size=4194304k -object {"qom-type":"memory-backend-ram","id":"pc.ram","size":4294967296} -overcommit mem-lock=off -smp 4,sockets=4,cores=1,threads=1 -uuid 6cbf3243-78a1-4daa-816a-d9702c143735 -no-user-config -nodefaults -chardev socket,id=charmonitor,fd=30,server=on,wait=off -mon chardev=charmonitor,id=monitor,mode=control -rtc base=localtime,driftfix=slew -global kvm-pit.lost_tick_policy=delay -no-shutdown -global ICH9-LPC.disable_s3=1 -global ICH9-LPC.disable_s4=1 -boot strict=on -device {"driver":"pcie-root-port","port":16,"chassis":1,"id":"pci.1","bus":"pcie.0","multifunction":true,"addr":"0x2"} -device {"driver":"pcie-root-port","port":17,"chassis":2,"id":"pci.2","bus":"pcie.0","addr":"0x2.0x1"} -device {"driver":"pcie-root-port","port":18,"chassis":3,"id":"pci.3","bus":"pcie.0","addr":"0x2.0x2"} -device {"driver":"pcie-root-port","port":19,"chassis":4,"id":"pci.4","bus":"pcie.0","addr":"0x2.0x3"} -device {"driver":"pcie-root-port","port":20,"chassis":5,"id":"pci.5","bus":"pcie.0","addr":"0x2.0x4"} -device {"driver":"pcie-root-port","port":21,"chassis":6,"id":"pci.6","bus":"pcie.0","addr":"0x2.0x5"} -device {"driver":"pcie-root-port","port":22,"chassis":7,"id":"pci.7","bus":"pcie.0","addr":"0x2.0x6"} -device {"driver":"pcie-root-port","port":23,"chassis":8,"id":"pci.8","bus":"pcie.0","addr":"0x2.0x7"} -device {"driver":"pcie-root-port","port":24,"chassis":9,"id":"pci.9","bus":"pcie.0","multifunction":true,"addr":"0x3"} -device {"driver":"pcie-root-port","port":25,"chassis":10,"id":"pci.10","bus":"pcie.0","addr":"0x3.0x1"} -device {"driver":"pcie-root-port","port":26,"chassis":11,"id":"pci.11","bus":"pcie.0","addr":"0x3.0x2"} -device {"driver":"pcie-root-port","port":27,"chassis":12,"id":"pci.12","bus":"pcie.0","addr":"0x3.0x3"} -device {"driver":"pcie-root-port","port":28,"chassis":13,"id":"pci.13","bus":"pcie.0","addr":"0x3.0x4"} -device {"driver":"pcie-root-port","port":29,"chassis":14,"id":"pci.14","bus":"pcie.0","addr":"0x3.0x5"} -device {"driver":"ich9-usb-ehci1","id":"usb","bus":"pcie.0","addr":"0x1d.0x7"} -device {"driver":"ich9-usb-uhci1","masterbus":"usb.0","firstport":0,"bus":"pcie.0","multifunction":true,"addr":"0x1d"} -device {"driver":"ich9-usb-uhci2","masterbus":"usb.0","firstport":2,"bus":"pcie.0","addr":"0x1d.0x1"} -device {"driver":"ich9-usb-uhci3","masterbus":"usb.0","firstport":4,"bus":"pcie.0","addr":"0x1d.0x2"} -device {"driver":"virtio-serial-pci","id":"virtio-serial0","bus":"pci.2","addr":"0x0"} -blockdev {"driver":"file","filename":"/var/lib/libvirt/images/win7_bios_q35.qcow2","node-name":"libvirt-2-storage","auto-read-only":true,"discard":"unmap"} -blockdev {"node-name":"libvirt-2-format","read-only":false,"driver":"qcow2","file":"libvirt-2-storage","backing":null} -device {"driver":"ide-hd","bus":"ide.0","drive":"libvirt-2-format","id":"sata0-0-0","bootindex":1} -blockdev {"driver":"file","filename":"/var/lib/libvirt/images/Win7_hygon.iso","node-name":"libvirt-1-storage","auto-read-only":true,"discard":"unmap"} -blockdev {"node-name":"libvirt-1-format","read-only":true,"driver":"raw","file":"libvirt-1-storage"} -device {"driver":"ide-cd","bus":"ide.1","drive":"libvirt-1-format","id":"sata0-0-1"} -netdev {"type":"tap","fd":"31","id":"hostnet0"} -device {"driver":"e1000e","netdev":"hostnet0","id":"net0","mac":"52:54:00:91:60:61","bus":"pci.1","addr":"0x0"} -chardev pty,id=charserial0 -device {"driver":"isa-serial","chardev":"charserial0","id":"serial0","index":0} -chardev spicevmc,id=charchannel0,name=vdagent -device {"driver":"virtserialport","bus":"virtio-serial0.0","nr":1,"chardev":"charchannel0","id":"channel0","name":"com.redhat.spice.0"} -device {"driver":"usb-tablet","id":"input0","bus":"usb.0","port":"1"} -audiodev {"id":"audio1","driver":"spice"} -spice port=5900,addr=127.0.0.1,disable-ticketing=on,image-compression=off,seamless-migration=on -device {"driver":"ich9-intel-hda","id":"sound0","bus":"pcie.0","addr":"0x1b"} -device {"driver":"hda-duplex","id":"sound0-codec0","bus":"sound0.0","cad":0,"audiodev":"audio1"} -global ICH9-LPC.noreboot=off -watchdog-action reset -chardev spicevmc,id=charredir0,name=usbredir -device {"driver":"usb-redir","chardev":"charredir0","id":"redir0","bus":"usb.0","port":"2"} -chardev spicevmc,id=charredir1,name=usbredir -device {"driver":"usb-redir","chardev":"charredir1","id":"redir1","bus":"usb.0","port":"3"} -device {"driver":"vfio-pci","host":"0000:06:00.0","id":"hostdev0","bus":"pci.4","addr":"0x0"} -device {"driver":"vfio-pci","host":"0000:06:00.1","id":"hostdev1","bus":"pci.5","addr":"0x0"} -device {"driver":"usb-host","hostdevice":"/dev/bus/usb/005/004","id":"hostdev2","bus":"usb.0","port":"4"} -device {"driver":"usb-host","hostdevice":"/dev/bus/usb/005/005","id":"hostdev3","bus":"usb.0","port":"5"} -device {"driver":"virtio-balloon-pci","id":"balloon0","bus":"pci.3","addr":"0x0"} -sandbox on,obsolete=deny,elevateprivileges=deny,spawn=deny,resourcecontrol=deny -msg timestamp=on
```
Added following items:    

```
<domain type='kvm' id='1' xmlns:qemu='http://libvirt.org/schemas/domain/qemu/1.0'>
......
  </seclabel>
  <qemu:override>
    <qemu:device alias='hostdev0'>
      <qemu:frontend>
        <qemu:property name='x-vga' type='bool' value='true'/>
      </qemu:frontend>
    </qemu:device>
  </qemu:override>
</domain>
```
