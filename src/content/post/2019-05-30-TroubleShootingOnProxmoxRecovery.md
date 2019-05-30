+++
title = "TroubleShootingOnPromoxIssue"
date = "2019-05-30T16:52:31+08:00"
description = "TroubleShootingOnPromoxIssue"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Problem
2 months ago I set a proxmox environment for a team which used as a dev
environment, for quickly snapshot and migration I choose zfs for the
filesystem, the structure is listed as following:    

![/images/2019_05_30_16_54_05_656x311.jpg](/images/2019_05_30_16_54_05_656x311.jpg)

But this environment got very slow i/o speed, I find the reason is because zfs
shouldn't rely on raid card.    

Which worse is: today after reboot, the grub runs into grub rescue:    

![/images/2019_05_30_16_55_53_385x62.jpg](/images/2019_05_30_16_55_53_385x62.jpg)

but you could not reached the /boot folder:    

![/images/2019_05_30_16_56_21_319x73.jpg](/images/2019_05_30_16_56_21_319x73.jpg)

### Solution
Raid Card configuration, change 01:22 and 01:23 from hotspare to raid1:    

![/images/2019_05_30_16_59_58_484x180.jpg](/images/2019_05_30_16_59_58_484x180.jpg)

to:    

![/images/2019_05_30_17_00_11_495x152.jpg](/images/2019_05_30_17_00_11_495x152.jpg)

Create new VD:   

![/images/2019_05_30_17_00_41_561x278.jpg](/images/2019_05_30_17_00_41_561x278.jpg)

Select these 2 disks:    

![/images/2019_05_30_17_00_55_686x337.jpg](/images/2019_05_30_17_00_55_686x337.jpg)

Select boot device(notice Boot device):    

![/images/2019_05_30_17_01_45_703x397.jpg](/images/2019_05_30_17_01_45_703x397.jpg)

Now Reinstall proxmox to VD 5.    

### System Configuration
Remove the `data` lv:    

```
# lvremove /dev/mapper/data
```
Create a new lv via:    

```
# lvcreate -n lv_root -L 150G pve
# mkfs.ext4 /dev/mapper/pve-lv_root
```
Now mount the zfs pools via:    

```
# zpool import -a
# zpool status
# zfs create -o canmount=noauto =o mountpoint=/mnt rpool/pve....
# zfs mount rpool/pve.....
```
Create new mount point and copy the old system into new:    

```
# mkdir /mnt1
# mount /dev/mapper/pve-lv_root /mnt1
# cp -arp /mnt/* /mnt1/
```
Now you have to change the grub.cfg:    

```
# vim /boot/grub/grub.cfg
if [ "${next_entry}" ] ; then
   set default="${next_entry}"
   set next_entry=
   save_env next_entry
   set boot_once=true
else
   set default="2"
fi



menuentry "Our Proxmox Boot Recovery" {
	load_video
	insmod gzio
	if [ x$grub_platform = xxen ]; then insmod xzio; insmod lzopio; fi
	insmod part_gpt
	insmod lvm
	insmod ext2
	set root='lvmid/ySIDEN-2G0X-DU6A-p0q8-cXsW-o6ja-IyhXuc/0M8yUI-yNQJ-Ntx8-8cfE-3g9k-0sOb-SDWfQe'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint='lvmid/ySIDEN-2G0X-DU6A-p0q8-cXsW-o6ja-IyhXuc/0M8yUI-yNQJ-Ntx8-8cfE-3g9k-0sOb-SDWfQe'  4fb86e38-eeae-
489a-b45e-3e5cc8055654
	else
	  search --no-floppy --fs-uuid --set=root 4fb86e38-eeae-489a-b45e-3e5cc8055654
	fi
	echo	'Loading Linux 4.15.17-1-pve ...'
	linux	/boot/vmlinuz-4.15.17-1-pve root=/dev/mapper/pve-lv_root ro  quiet
	echo	'Loading initial ramdisk ...'
	initrd	/boot/initrd.img-4.15.17-1-pve
}
menuentry "Memory test (memtest86+)" {
	insmod part_gpt
```
Edit the fstab:    

```
$ vim /mnt1/etc/fstab 
# <file system> <mount point> <type> <options> <dump> <pass>
/dev/pve/lv_root / ext4 errors=remount-ro 0 1
/dev/pve/swap none swap sw 0 0
proc /proc proc defaults 0 0
```
Now reboot the system you will get into the newly-created lvm based system,
and run an ext4 based OS which acts the same as the old ones.    
