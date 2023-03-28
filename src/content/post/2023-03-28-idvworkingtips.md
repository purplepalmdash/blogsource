+++
title= "idvworkingtips"
date = "2023-03-28T08:19:43+08:00"
description = "idvworkingtips"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 0. Hardware
![/images/2023_03_28_08_27_44_715x718.jpg](/images/2023_03_28_08_27_44_715x718.jpg)

Detailed info:   

```
11th Gen Intel(R) Core(TM) i7-1165G7 @ 2.80GHz
16GB DDR4-3200(dual channel)
500GB PCIe4.0 ssd
```
### 1. System Installation

Downloads iso and write to flash disk(Above 8G):    

```
wget https://mirrors.ustc.edu.cn/rocky/9.1/isos/x86_64/Rocky-9.1-20221214.1-x86_64-dvd.iso
sudo dd if=./Rocky-9.1-20221214.1-x86_64-dvd.iso of=/dev/sdb bs=1M && sudo sync
```
Install RockLinux 9.1 to nuc.   

Installation configurations:   

```
Select English->English(United States)
Set user passwd(Allow root SSH login with password)
Software Selection(Minimal Install)
Partition: Delete /home, shrink swap to 2GiB(example layout----/boot: 1024MiB, /:462.26GiB,/boot/efi: 512MiB, swap: 2GiB)
Create user: idv(make this user to be administrator)
Set hostname: rockyIDV
```
After installation, change default repo configuration:    

```
$ sudo su
# sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g' \
    -i.bak \
    /etc/yum.repos.d/rocky-extras.repo \
    /etc/yum.repos.d/rocky.repo
# dnf makecache
```
Instal mate desktop enviroment as the idv graphical UI:    

```
sudo dnf config-manager --set-enabled crb
sudo dnf install epel-release
sudo dnf install NetworkManager-adsl NetworkManager-bluetooth NetworkManager-libreswan-gnome NetworkManager-openvpn-gnome NetworkManager-ovs NetworkManager-ppp NetworkManager-team NetworkManager-wifi NetworkManager-wwan adwaita-gtk2-theme alsa-plugins-pulseaudio atril atril-caja atril-thumbnailer caja caja-actions caja-image-converter caja-open-terminal caja-sendto caja-wallpaper caja-xattr-tags dconf-editor engrampa eom firewall-config gnome-disk-utility gnome-epub-thumbnailer gstreamer1-plugins-ugly-free gtk2-engines gucharmap gvfs-fuse gvfs-gphoto2 gvfs-mtp gvfs-smb initial-setup-gui libmatekbd libmatemixer libmateweather libsecret lm_sensors marco mate-applets mate-backgrounds mate-calc mate-control-center mate-desktop mate-dictionary mate-disk-usage-analyzer mate-icon-theme mate-media mate-menus mate-menus-preferences-category-menu mate-notification-daemon mate-panel mate-polkit mate-power-manager mate-screensaver mate-screenshot mate-search-tool mate-session-manager mate-settings-daemon mate-system-log mate-system-monitor mate-terminal mate-themes mate-user-admin mate-user-guide mozo network-manager-applet nm-connection-editor p7zip p7zip-plugins pluma seahorse seahorse-caja xdg-user-dirs-gtk pciutils vim
sudo dnf install lightdm-settings lightdm
sudo systemctl set-default graphical.target
sudo reboot
```
Enable the lightdm autologin:    

```
sudo nano /etc/lightdm/lightdm.conf
autologin-user=idv
```
### 2. Virtualization Configuration
Install and configure libvirtd:    

```
sudo dnf install libvirt-daemon qemu-kvm virt-top libguestfs-tools virt-install
sudo systemctl enable libvirtd
sudo virsh net-start default
```
view qemu version:     

```
# /usr/libexec/qemu-kvm --version
QEMU emulator version 7.0.0 (qemu-kvm-7.0.0-13.el9_1.2)
Copyright (c) 2003-2022 Fabrice Bellard and the QEMU Project developers
```
Kernel default options:     

```
# vi /etc/default/grub
......
GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on intel_iommu=pt kvm.ignore_msrs=1"
# grub2-mkconfig -o /boot/efi/EFI/rocky/grub.cfg
# cat /proc/cmdline
Examine the iommu
# sudo dmesg | grep iommu
# sudo dmesg | grep -i dmar
```
Edit dracut for adding vfio modules:    

```
[idv@rockyIDV ~]$ sudo vim /etc/dracut.conf.d/vfio.conf
add_drivers+="vfio vfio_iommu_type1 vfio_pci vfio_virqfd"
[idv@rockyIDV ~]$ sudo dracut -f --kver  `uname -r`
[idv@rockyIDV ~]$ sudo reboot
```
### 3. libvirtd Hooks
Refers to other material.   

### 4. issue
On rocky linux, we got following issues:    

```
[idv@rockyIDV ~]$ ./test_10thFV_win10.sh 
2023-03-28T05:53:46.994085Z qemu-system-x86_64: -device vfio-pci,host=0000:00:02.0,x-igd-gms=2,id=hostdev0,bus=pci.0,addr=2,rombar=0,x-igd-opregion=on: IGD device 0000:00:02.0 does not support OpRegion access,legacy mode disabled
2023-03-28T05:53:46.995006Z qemu-system-x86_64: -device vfio-pci,host=0000:00:02.0,x-igd-gms=2,id=hostdev0,bus=pci.0,addr=2,rombar=0,x-igd-opregion=on: vfio 0000:00:02.0: does not support requested IGD OpRegion feature: No such device

```
Now change back to Ubuntu22.04.     

Also spice-server is removed for rhel9, so got lots of time for building spice related libs.   
