+++
title= "ArchInstallOnUSB"
date = "2021-01-05T14:01:34+08:00"
description = "ArchInstallOnUSB"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Hardware
320G usb disk, laptop(running archlinux already).   
### Steps
`fdisk` the usb disk and create with following partitions:    

```
$ sudo fdisk -l /dev/sdc
Disk /dev/sdc：298.09 GiB，320072933376 字节，625142448 个扇区
磁盘型号：Storage         
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x112a2f3d

设备       启动    起点      末尾      扇区   大小 Id 类型
/dev/sdc1          2048   1050623   1048576   512M ef EFI (FAT-12/16/32)
/dev/sdc2       1050624 625142447 624091824 297.6G 83 Linux

```
Format the disk:    

```
$ $ sudo mkfs.fat -F32 /dev/sdc1
mkfs.fat 4.1 (2017-01-24)
$ sudo mkfs.ext4 /dev/sdc2
```
Install `arch-install-scripts` on archlinux. Then mount the disk to install point:   

```
$ sudo mount /dev/sdc2 /mnt
$ sudo mkdir -p /mnt/boot
$ sudo mount /dev/sdc1 /mnt/boot
```
Now use `pacstrap` for installing basic system onto usb disk:   

```
$ sudo pacstrap -c /mnt base linux linux-firmware base-devel
```
Generate `/etc/fstab`:    

```
# genfstab -U /mnt >> /mnt/etc/fstab
# vim /mnt/etc/fstab
comment the swap partition
```
chroot into /mnt:    

```
# arch-chroot /mnt
``` 

```
# ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# pacman -S vim
# vim /etc/locale.gen
en_US.UTF-8 UTF-8  
en_US ISO-8859-1  
zh_CN.GB18030 GB18030  
zh_CN.GBK GBK  
zh_CN.UTF-8 UTF-8  
zh_CN GB2312 
# locale-gen
# vim /etc/locale.conf
LANG=en_US.UTF-8
# vim /etc/hostname
archusb
# vim /etc/hosts 
    # Static table lookup for hostnames.
    # See hosts(5) for details.
    127.0.0.1	localhost
    ::1		localhost
    127.0.1.1	archusb
# pacman -S net-tools tcpdump iotop dhcpcd openssh dosfstools ntfs-3g amd-ucode intel-ucode grub
# systemctl enable sshd
# cat /etc/mkinitcpio.conf | grep block
    #    HOOKS=(base udev autodetect block filesystems)
    #    HOOKS=(base udev block filesystems)
    #    HOOKS=(base udev block mdadm encrypt filesystems)
    #    HOOKS=(base udev block lvm2 filesystems)
    HOOKS=(base udev block keyboard autodetect modconf filesystems fsck)
#  mkinitcpio -P
# passwd
```
Make grub configuration:    

```
# grub-install --target=i386-pc /dev/sdc --recheck
# grub-install --target=x86_64-efi --efi-directory=/boot/efi --removable --recheck
```
Support generic gpu:    

```
# pacman -S xf86-video-vesa xf86-video-ati xf86-video-intel xf86-video-amdgpu xf86-video-nouveau xf86-video-fbdev
```
Network configuration:    

```
# pacman -S networkmanager
# systemctl enable NetworkManager
# grub-mkconfig -o /boot/grub/grub.cfg

```
Now you could use usb disk for booting up the system, enjoy it.  

### libvirt configuration
Install iptables, etc.    

```
# pacman -S ebtables iptables dnsmasq
```
Configure bridge networking using network manager:     

```
$ nmcli connection add type bridge ifname br0 stp no
$ nmcli connection add type bridge-slave ifname enp30s0 master br0
```
Case static ip address:    

```
nmcli conn add type bridge ifname br0 ipv4.method manual ipv4.address "10.137.149.5/24" ipv4.gateway "10.137.149.1" ipv4.dns 223.5.5.5 
nmcli connection add type bridge-slave ifname eth0 master br0
```


iptables for libvirt:    

```
# iptables -I FORWARD -m physdev --physdev-is-bridged -j ACCEPT
# iptables-save -f /etc/iptables/iptables.rules
# systemctl enable iptables.service
```
Then your bridge could be use. 
