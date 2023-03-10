+++
title= "RecoveryFromUEFI"
date = "2023-03-10T15:57:59+08:00"
description = "RecoveryFromUEFI"
keywords = ["Technology"]
categories = ["Technology"]
+++
Use archlinux installation disk for entering the command line, then:     

```
mkdir /mnt/arch
mount -t auto /dev/sda2 /mnt/arch
mount -t auto /dev/sda1 /mnt/arch/boot/
arch-chroot /mnt/arch
```

Re-install grub:    

```
grub-install --efi-directory=/boot/efi --target=x86_64-efi /dev/sda
```
Then reboot the machine we could see the archlinux reback again.   
