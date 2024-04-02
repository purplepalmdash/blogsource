+++
title= "CreatingFOGUbuntu2204UEFIImage"
date = "2024-04-02T14:40:24+08:00"
description = "CreatingFOGUbuntu2204UEFIImage"
keywords = ["Technology"]
categories = ["Technology"]
+++
Configuration:     

![/images/2024_04_02_14_40_40_489x210.jpg](/images/2024_04_02_14_40_40_489x210.jpg)

After installation, do following:     

```
# scp test@xxxxxx.xxx/grubdebs .
# ls grubdebs
grub2-common_2.06-13+deb12u1_amd64.deb
grub-common_2.06-13+deb12u1_amd64.deb
grub-efi-amd64_2.06-13+deb12u1_amd64.deb
grub-efi-amd64-bin_2.06-13+deb12u1_amd64.deb
grub-efi-amd64-signed_1+2.06+13+deb12u1_amd64.deb
grub-pc_2.06-13+deb12u1_amd64.deb
grub-pc-bin_2.06-13+deb12u1_amd64.deb
install.sh
libfuse2_2.9.9-6+b1_amd64.deb
shim-helpers-amd64-signed_1+15.7+1_amd64.deb
shim-signed_1.39+15.7-1_amd64.deb
shim-signed-common_1.39+15.7-1_all.deb
shim-unsigned_15.7-1_amd64.deb
# mv grub-pc* ../
# dpkg -i *.deb
# cd ..
# dpkg -i *.deb
```
hold the installed packages:    

```
# apt-mark hold grub-common grub-efi-amd64 grub-efi-amd64-bin grub-efi-amd64-signed grub-pc grub-pc-bin grub2-common libfuse2 shim-helpers-amd64-signed shim-signed:amd64 shim-signed-common shim-unsigned
```
Reinstall the grub to let the new package take effect:     

```

sudo umount /boot/efi
sudo mkfs.vfat -F32 /dev/vda1
sudo mount /dev/vda1 /boot/efi
sudo update-grub
sudo update-grub2
sudo grub-install /dev/vda
sudo grub2-mkconfig -o /boot/efi/EFI/ubuntu/grub.cfg
vim /etc/fstab
change efi 
sudo reboot
```
Install fog-client(ubuntu2204):    

```
sudo apt update
sudo apt install nuget
sudo apt install mono-complete
sudo apt install apt-transport-https
sudo mono SmartInstaller.exe
```
![/images/2024_04_02_15_11_19_782x549.jpg](/images/2024_04_02_15_11_19_782x549.jpg)
Now shutdown the machine, Change to pxe mode, to test its start-up.  

![/images/2024_04_02_15_14_42_464x247.jpg](/images/2024_04_02_15_14_42_464x247.jpg)

Registeration image:    

![/images/2024_04_02_15_57_50_887x487.jpg](/images/2024_04_02_15_57_50_887x487.jpg)

Associate the image with newly created image:    

![/images/2024_04_02_15_57_19_1074x461.jpg](/images/2024_04_02_15_57_19_1074x461.jpg)

Capture the image from this node:     

![/images/2024_04_02_15_58_22_943x398.jpg](/images/2024_04_02_15_58_22_943x398.jpg)

partclone and upload the image:     

![/images/2024_04_02_16_00_17_558x372.jpg](/images/2024_04_02_16_00_17_558x372.jpg)
 
