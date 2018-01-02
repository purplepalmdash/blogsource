+++
date = "2017-01-24T15:35:05+08:00"
categories = ["Linux"]
keywords = ["Encryption"]
description = "Encrypt your partition using usb key"
title = "CentOS Encryption"

+++
### Disk Configuration
Specify the disk like following, notice we don't activate the swap partition.    

Select the disk and do the following configuration:    

![/images/2017_01_24_10_19_50_492x504.jpg](/images/2017_01_24_10_19_50_492x504.jpg)    

Boot partition, should be 400MiB:    

![/images/2017_01_24_19_10_57_688x499.jpg](/images/2017_01_24_19_10_57_688x499.jpg)    

Root partition, should contains all of the left space:    

![/images/2017_01_24_19_11_21_654x422.jpg](/images/2017_01_24_19_11_21_654x422.jpg)    

### Cryption
Refers to:    

[http://www.gaztronics.net/howtos/luks.php](http://www.gaztronics.net/howtos/luks.php)    

Generate the secret key file:    

```
# dd if=/dev/sdb of=luks-secret.key bs=1 count=4096
# cryptsetup luksAddKey /dev/sda2 luks-secret.key --key-slot 1
```
Modify Dracut:    

```
# ls  /dev/disk/by-id | grep usb
usb-SanDisk_Cruzer_Orbit_4C532000050606114400-0:0 -> ../../sda
usb-SanDisk_Cruzer_Orbit_4C532000050606114400-0:0-part1 -> ../../sda1
```
Remember the id of sda, later we will use it for updating `cryptroot-ask.sh`
file:    

```
# vim /usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh
info "luksOpen $device $luksname $luksfile $luksoptions"

+++ # Unlock with USB key
+++ sleep 3
+++ udevsettle
+++ usbkey=/dev/disk/by-id/usb-SanDisk_Cruzer_Orbit_4C532000050606114400-0\:0
+++ if [ -e $usbkey ]; then
+++   ask_passphrase=0
+++   echo "USB Key detected - unlocking partition $device ..."
+++   dd if=$usbkey bs=1 count=4096 | cryptsetup luksOpen $device $luksname --key-file=-
+++ fi


OLD_IFS="$IFS"
```
Notice the prefixed `+++` is the added lines to this file.     

Also because the program `dd` is not loaded by default in CentOS, we need add 
following line into `/usr/lib/dracut/modules.d/90crypt/module-setup.sh`:    

```
# vim /usr/lib/dracut/modules.d/90crypt/module-setup.sh
install() {
    <snip>
    dracut_need_initqueue
+++    inst /usr/bin/dd
}
```
We need to modify `/etc/dracut.conf` to stop systemd from running the decryption 
process:    

```
# dracut modules to omit
omit_dracutmodules+="systemd"

# dracut modules to add to the default
add_dracutmodules+="crypt"
```
Create the new initramfs via `dracut -f`, now insert the usb disk, the decryption 
will be automatically executed, enjoy your own usb disk encrypted system!    

### Hacking
For automatically dracut after system upgrading:    

```
# mkdir -p /root/archive
# mkdir -p /root/scripts
# cp /usr/lib/dracut/modules.d/90crypt/cryptroot-ask.sh  /root/archive
# cp /usr/lib/dracut/modules.d/90crypt/module-setup.sh /root/archive
# vim /root/scripts/update-dracut
    #!/bin/bash
    cp /root/archive/cryptroot-ask.sh /usr/share/dracut/modules.d/90crypt/
    cp /root/archive/module-setup.sh /usr/share/dracut/modules.d/90crypt/
    dracut -f
    exit 0
# chmod 777 /root/scripts/update-dracut
```

### Disable Kernel Update
For we make the updates to initramfs, we'd better disable kernel update for CentOS.    

```
$ sudo vim /etc/yum.conf
exclude=kernel*
```
Now via `yum update` you won't get your kernel related packages update.    

### LVM Based Encryption
If you use lvm based, then do following things:    

```
# cat /etc/crypttab 
luks-189e8c45-2c62-4c08-acb6-b3264c435fd1 UUID=189e8c45-2c62-4c08-acb6-b3264c435fd1 none 
# blkid
/dev/sda1: LABEL="XENSERVER-6" UUID="5CD6-02A1" TYPE="vfat" 
/dev/sdb1: UUID="d13618c7-b166-4135-8cae-1c5b8c5110fc" TYPE="xfs" 
/dev/sdb2: UUID="tZyuzH-wIpt-MtQv-gWsJ-Ld69-zjtQ-TfbL6z" TYPE="LVM2_member" 
/dev/mapper/cl-00: UUID="189e8c45-2c62-4c08-acb6-b3264c435fd1" TYPE="crypto_LUKS" 
/dev/mapper/luks-189e8c45-2c62-4c08-acb6-b3264c435fd1: UUID="f7a6d197-5afd-46e1-8a69-408de5278405" TYPE="xfs" 

```
Then your command should be like:    

```
# cryptsetup luksAddKey /dev/mapper/cl-00 luks-secret.key --key-slot 1
```
Make sure the partition is the same as the one you looked as `crypto_LUKS`.    
