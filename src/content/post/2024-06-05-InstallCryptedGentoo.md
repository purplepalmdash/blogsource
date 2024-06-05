+++
title= "InstallCryptedGentoo"
date = "2024-06-05T09:46:02+08:00"
description = "InstallCryptedGentoo"
keywords = ["Technology"]
categories = ["Technology"]
+++
Partition:    

```
livecd ~ # lsblk
NAME  MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0   7:0    0 479.7M  1 loop /mnt/livecd
sr0    11:0    1 527.4M  0 rom  /mnt/cdrom
vda   252:0    0    80G  0 disk 
zram0 253:0    0     0B  0 disk 
livecd ~ # parted /dev/vda
GNU Parted 3.6
Using /dev/vda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) mklabel gpt
(parted) unit MiB                                                         
(parted) mkpart primary 2 514
(parted) mkpart primary 515 -1                                            
(parted) name 1 boot                                                      
(parted) name 2 luks                                                      
(parted) set 1 boot on                                                    
(parted) q                                                                
Information: You may need to update /etc/fstab.
```
Setup crypts:    

```
livecd ~ # cryptsetup luksFormat  /dev/vda2

WARNING!
========
This will overwrite data on /dev/vda2 irrevocably.

Are you sure? (Type 'yes' in capital letters): YES
Enter passphrase for /dev/vda2: 
Verify passphrase: 
livecd ~ # cryptsetup open /dev/vda2 ct0
Enter passphrase for /dev/vda2: 
```
Mount to specified position:     

```
mkfs.btrfs /dev/mapper/ct0
mkfs.vfat -F32 /dev/vda1
mount /dev/mapper/ct0 /mnt/gentoo
```
Using btrfs's subvolume function:     

```
btrfs subvolume create /mnt/gentoo/subvol-root
btrfs subvolume create /mnt/gentoo/subvol-home
btrfs subvolume create /mnt/gentoo/subvol-snapshots
btrfs subvolume set-default /mnt/gentoo/subvol-root
umount /mnt/gentoo
mount /dev/mapper/ct0 /mnt/gentoo
```
prepare chroot:     

```
cd /mnt/gentoo
wget ......stage3
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```
specify jobs for building:      

```
# nano /etc/portage/make.conf
MAKEOPTS="-j17"
```
select mirror:     

```
mirrorselect -i -o >>/mnt/gentoo/etc/portage/make.conf
```
Setup ebuild repository sync address:      

```
# mkdir etc/portage/repos.conf
# cp usr/share/portage/config/repos.conf etc/portage/repos.conf/gentoo.conf
# cat etc/portage/repos.conf/gentoo.conf | grep uri
sync-uri = rsync://rsync.mirrors.ustc.edu.cn/gentoo-portage
cp -L /etc/resolv.conf etc/
```
chroot-in:    

```
livecd /mnt/gentoo # mount --types proc /proc /mnt/gentoo/proc
livecd /mnt/gentoo # mount --rbind /sys /mnt/gentoo/sys
livecd /mnt/gentoo # mount --rbind /dev /mnt/gentoo/dev
####  then
chroot /mnt/gentoo
. /etc/profile
PS1=(chroot)$PS1
```
Mount vda1 for kernel/bootloader installation:      

```
mount /dev/vda1 /boot
```
Install gentoo ebuild repository:      

```
emerge-webrsync
emerge --sync 
```
Install and set editor:     

```
emerge -vj app-editors/vim
eselect editor set 2
. /etc/profile
PS1=(chroot)$PS1
```
update @world:    

```
eselect profile set 21
USE="X initramfs cjk cups crypt udev alsa elogind zsh-completion bash-completion -consolekit"
emerge --ask --verbose --update --deep --changed-use @world
```
Select the locale and timezone:        

```
echo "Asia/Shanghai" > /etc/timezone
emerge --config sys-libs/timezone-data
vim /etc/locale.gen
locale-gen 
eselect locale set 6
env-update && source /etc/profile && export PS1="(chroot) $PS1"
```

Install firmware:     

```
mkdir -p /etc/portage/package.license
echo 'sys-kernel/linux-firmware linux-fw-redistributable no-source-code' >/etc/portage/package.license/linux-firmware
echo 'sys-kernel/installkernel dracut' >/etc/portage/package.use/installkernel
emerge --ask sys-kernel/gentoo-sources
emerge --ask sys-kernel/linux-firmware
emerge --ask sys-apps/pciutils
emerge --ask sys-kernel/genkernel
```
Set kernel:     

```
(chroot) livecd / # readlink -v /usr/src/linux
readlink: /usr/src/linux: No such file or directory
(chroot) livecd / # eselect kernel list
Available kernel symlink targets:
  [1]   linux-6.6.30-gentoo
(chroot) livecd / # eselect kernel set 1
(chroot) livecd / # readlink -v /usr/src/linux
linux-6.6.30-gentoo
```
Build kernel:      

```
cd /usr/src/linux
 make menuconfig
 make -j17
 make modules_install && make install
 genkernel --kernel-config=/usr/src/linux/.config initramfs
 blkid /dev/vda1>>/etc/fstab
 blkid /dev/mapper/ct0>>/etc/fstab
 vim /etc/fstab 
#/dev/vda1: UUID="3D3E-221D" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="boot" PARTUUID="e0e7d44b-d78f-4c27-808a-c859ce8ead64"
UUID="3D3E-221D"	/boot     vfat    rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=iso8859-1,shortname=mixed,errors=remount-ro 0 2
#/dev/mapper/ct0: UUID="5f574106-322e-4f9f-8efd-c3615fcb237a" UUID_SUB="0e96210f-49ac-4355-9237-dab1fb6eae93" BLOCK_SIZE="4096" TYPE="btrfs"
# rw,relatime,space_cache=v2,subvolid=256,subvol=/subvol-root
UUID="5f574106-322e-4f9f-8efd-c3615fcb237a"	/         btrfs   defaults,noatime,ssd,discard,subvolid=256,subvol=/subvol_root 0 1

```
Install system packages:      
 
```
   66  emerge --ask sys-fs/cryptsetup
   67  emerge --ask sys-process/cronie
   68  emerge --ask app-admin/sysklogd
   69  emerge --ask sysfs/btrfs-progs
   70  emerge --ask sys-fs/btrfs-progs
   71  emerge --ask net-misc/dhcpcd
   72  emerge -av app-admin/sysklogd sys-fs/cryptsetup
   73  rc-update add sysklogd default
   74  rc-update add dhcpcd default
   76  echo GRUB_PLATFORMS="efi-64" >> /etc/portage/make.conf
   77  emerge -av sys-boot/grub:2
   79  mkdir /boot/efi/
   80  ls /dev/disk/by-uuid/
   81  ls /dev/disk/by-uuid/ -l -h
   82  cat /etc/fstab 
   83  vim /etc/default/grub 
   84  vim /etc/default/grub 
   85  mount -a
   86  grub-install --target=x86_64-efi --boot-directory=/boot --efi-directory=/boot/efi/ --bootloader-id=Gentoo --debug
   87  grub-mkconfig -o /boot/grub/grub.cfg
   89  ls /boot/grub/
   90  ls /boot/grub/grub.cfg 
```
