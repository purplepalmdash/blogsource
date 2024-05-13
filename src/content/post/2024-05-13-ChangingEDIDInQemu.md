+++
title= "ChangingEDIDInQemu"
date = "2024-05-13T10:25:34+08:00"
description = "ChangingEDIDInQemu"
keywords = ["Technology"]
categories = ["Technology"]
+++
### edid retrieve
Install edid related packages:    

```
$ sudo apt install -y edid-decode read-edid
```
Find the edid in /sys:    

```
$ sudo find /sys | grep edid
/sys/kernel/debug/dri/0/Virtual-1/edid_override
/sys/devices/pci0000:00/0000:00:01.0/drm/card0/card0-Virtual-1/edid
/sys/module/drm_kms_helper/parameters/edid_firmware
/sys/module/drm/parameters/edid_firmware
/sys/module/drm/parameters/edid_fixup

```
The  `/sys/devices/pci0000:00/0000:00:01.0/drm/card0/card0-Virtual-1/edid` file is your edid file, transfer it to windows machine.   

### edid modification

Using AW EDID Editor for editing the edid file:    

![/images/2024_05_13_10_26_13_772x524.jpg](/images/2024_05_13_10_26_13_772x524.jpg)

Remove the items you won't preserve:    

![/images/2024_05_13_10_26_41_834x651.jpg](/images/2024_05_13_10_26_41_834x651.jpg)

Modified items:    

![/images/2024_05_13_10_27_10_554x397.jpg](/images/2024_05_13_10_27_10_554x397.jpg)

### Using the new edid file
Edit the grub options:    

```
$ sudo vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet drm.edid_firmware=Virtual-1:edid/edidno1080.bin"
```
Add initram-tools hooks(Added scripts in kmod - the hacking way):    

```
$ cat /usr/share/initramfs-tools/hooks/kmod 

......
......
......
# hacking for edid
mkdir -p ${DESTDIR}/lib/firmware/edid
mkdir -p ${DESTDIR}/usr/lib/firmware/edid

cp /lib/firmware/edid/edidno4k.bin ${DESTDIR}/lib/firmware/edid/
cp /lib/firmware/edid/edidno1080.bin ${DESTDIR}/lib/firmware/edid/
cp /usr/lib/firmware/edid/edidno4k.bin ${DESTDIR}/usr/lib/firmware/edid/
cp /usr/lib/firmware/edid/edidno1080.bin ${DESTDIR}/usr/lib/firmware/edid/

```
Update the kernel and initramfs:    

```
$ sudo update-grub2
$ sudo update-initramfs -u -k all
```
### Effects
edid-decode result:    

```
root@idvrescue:/home/hhh# edid-decode < /sys/devices/pci0000:00/0000:00:01.0/drm/card0/card0-Virtual-1/edid
......
hecksum: 0x47

----------------

Block 1, CTA-861 Extension Block:
  Revision: 3
  Native detailed modes: 0
  Video Data Block:
    VIC  31:  1920x1080   50.000000 Hz  16:9     56.250 kHz    148.500000 MHz
Checksum: 0x95

```

![/images/2024_05_13_10_31_17_647x580.jpg](/images/2024_05_13_10_31_17_647x580.jpg)
