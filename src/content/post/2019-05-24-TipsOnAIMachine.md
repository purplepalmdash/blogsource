+++
title = "TipsOnAIMachine"
date = "2019-05-24T10:29:08+08:00"
description = "TipsOnAIMachine"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 硬件环境
新购的神舟， Z7-KP7GH， CPU， i7-8750H， 内存24G， 显卡Nvidia GTX1060 6G.      
8G 优盘用于系统安装.    
### 软件安装及适配
ubuntu-18.04.2-desktop-amd64.iso, 写入优盘:    

```
# sudo dd if=./ubuntu-18.04.2-desktop-amd64.iso of=/dev/sdc bs=1M && sudo sync
```
笔记本开机按DEL进入BIOS配置，选择U盘启动，遇到安装卡住的问题，解决方案如下：    

```
GRUB choose the Ubuntu, or Install Ubuntu (it depends, you will see it hopefully), go to it with the arrows and press the 'e' key.
Here go to the line which contains quiet splash at the end and add  acpi=off after these words.
Then press F10 to boot with these settings.
```

安装中需要重新分区, 参考：    

![/images/2019_05_24_10_41_45_771x380.jpg](/images/2019_05_24_10_41_45_771x380.jpg)
这里新建了efi分区，并使用新建的分区用于安装操作系统，同时保留了原有的Windows操作系统，特别要注意的是关于bootloader的安装位置。    

安装完毕后，由于是nvidia卡的原因，首次进入系统会卡住，这里我们需要再次修改GRUB进入系统:    

```
When you are in the GRUB menu, press E to enter the GRUB editor. Add nouveau.modeset=0 to the end of the line that starts with linux. After you've added it, press F10 to boot. Your system should start. After that, go to System Settings > Software & Updates > Additional Drivers and then select the NVIDIA driver. Right now I'm using NVIDIA binary driver- version 367.57 from nvidia-367 (proprietary, tested).
```
当前(2019-05-24)时，nvidia的驱动是nvida-driver-390.    

现在重新启动机器，就可以正常进入系统并执行操作了。    
显卡的测试可以参考[https://linuxconfig.org/benchmark-your-graphics-card-on-linux](https://linuxconfig.org/benchmark-your-graphics-card-on-linux)
时间的关系这里我就不做了。
### 系统适配
安装必要的包:    

```
# apt-get install -y openssh-server vim net-tools virt-manager vagrant
vagrant-libvirt
```

