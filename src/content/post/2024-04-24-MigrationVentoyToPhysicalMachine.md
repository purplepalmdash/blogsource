+++
title= "MigrationVentoyToPhysicalMachine"
date = "2024-04-24T14:50:34+08:00"
description = "MigrationVentoyToPhysicalMachine"
keywords = ["Technology"]
categories = ["Technology"]
+++
Two machines, one is verified vm, target machine is a physical machine(192.168.1.184), do following:    

on verified vm:    

```
mount /dev/sda2 /mnt8
cd /mnt8
scp -r HHHISO/ ventoy/ root@192.168.1.184:/mnt8/
cd /boot/efi
scp -r grub/ tool/ ventoy/ vtldr  root@192.168.1.184:/boot/efi/
cd /boot/efi/EFI
scp -r VENTOY/ root@192.168.1.184:/boot/efi/EFI/
scp /etc/grub.d/99_ventoy  root@192.168.1.184:/etc/grub.d/
```

on target physical machine, do following:     

```
# vim /etc/default/grub
GRUB_DEFAULT=0
GRUB_TIMEOUT_STYLE=menu
GRUB_TIMEOUT=3
# update-grub2
# grub-mkconfig -o /boot/grub/grub.cfg
# reboot
```
ventoy screenshot:   

![/images/2024_04_24_14_53_48_879x553.jpg](/images/2024_04_24_14_53_48_879x553.jpg)

win11 screenshot:    

![/images/2024_04_24_14_54_46_1577x908.jpg](/images/2024_04_24_14_54_46_1577x908.jpg)

win10 screenshot:    

![/images/2024_04_24_14_56_00_1716x1046.jpg](/images/2024_04_24_14_56_00_1716x1046.jpg)

