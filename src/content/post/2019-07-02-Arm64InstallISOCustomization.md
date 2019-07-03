+++
title = "Arm64ISOCustomization"
date = "2019-07-02T09:51:48+08:00"
description = "Arm64ISOCustomization"
keywords = ["Linux"]
categories = ["Linux"]
+++
Make working directory:     

```
#  mkdir Rong1907iso
# cd Rong1907iso/
# cp ../ubuntu-18.04.2-server-arm64.iso .
# cp -r ./iso/* ./newISO
# cp -r ./iso/.disk ./newISO
# umount ./iso
# rm -f ubuntu-18.04.2-server-arm64.iso 
# rm -rf iso/
```
Add seed files under `preseed` directory, then edit the grub files:   

```
root@arm02:/home/test/Rong1907iso/newISO# ls preseed/
cli.seed      hwe-ubuntu-server-minimal.seed    hwe-ubuntu-server.seed  fuck_auto.seed        ubuntu-server-minimal.seed    ubuntu-server.seed
hwe-cli.seed  hwe-ubuntu-server-minimalvm.seed  fuck.seed            fuck_auto_multi.seed  ubuntu-server-minimalvm.seed
root@arm02:/home/test/Rong1907iso/newISO# ls boot/grub/grub.cfg 
boot/grub/grub.cfg
```
Edit the grub file like following:    

```
set menu_color_normal=white/black
set menu_color_highlight=black/yellow

insmod gzio

set timeout=10
menuentry "Auto Install Ubuntu Server(Manual-Partition)" {
        set gfxpayload=keep
        linux   /install/vmlinuz auto-install/enable=true file=/cdrom/preseed/fuck.seed quiet ---
        initrd  /install/initrd.gz
}
menuentry "Auto Install Ubuntu Server(Auto-Partition-AllInOne)" {
        set gfxpayload=keep
        linux   /install/vmlinuz auto-install/enable=true file=/cdrom/preseed/fuck_auto.seed quiet ---
        initrd  /install/initrd.gz
}
menuentry "Auto Install Ubuntu Server(Auto-Partition-Seperate)" {
        set gfxpayload=keep
        linux   /install/vmlinuz auto-install/enable=true file=/cdrom/preseed/fuck_auto_multi.seed quiet ---
        initrd  /install/initrd.gz
}
menuentry "Install Ubuntu Server" {
        set gfxpayload=keep
        linux   /install/vmlinuz  file=/cdrom/preseed/ubuntu-server.seed quiet ---
        initrd  /install/initrd.gz
}

```
Make the iso via following command:    

```
# xorriso -as mkisofs -r -checksum_algorithm_iso md5,sha1 -V 'Server 18.04.2 LTS arm64' -o ./fuck_ubuntu180402_arm64.iso -J -joliet-long -cache-inodes -e boot/grub/efi.img  -no-emul-boot -append_partition 2 0xef newISO/boot/grub/efi.img  -partition_cyl_align all newISO/
root@arm02:/home/test/Rong1907iso# ls
newISO  fuck_ubuntu180402_arm64.iso
```
Using the `fuck_ubuntu180402_arm64.iso` you could install systme on arm64 based server.   
