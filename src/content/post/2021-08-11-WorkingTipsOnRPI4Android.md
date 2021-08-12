+++
title= "WorkingTipsOnRPI4Android"
date = "2021-08-11T17:20:50+08:00"
description = "WorkingTipsOnRPI4Android"
keywords = ["Technology"]
categories = ["Technology"]
+++
### flash
Create a new partition:   

![/images/2021_08_11_17_21_41_712x293.jpg](/images/2021_08_11_17_21_41_712x293.jpg)

Create new unkown partitions, then a userdata partition:   

![/images/2021_08_11_17_23_47_712x289.jpg](/images/2021_08_11_17_23_47_712x289.jpg)

Layout:    

![/images/2021_08_11_17_24_20_951x286.jpg](/images/2021_08_11_17_24_20_951x286.jpg)

```
p1: 128MB for boot (fat32, boot & lba)
p2: 768MB for /system
p3: 128MB for /vendor
p4: remainings for /data (ext4)
```

Manage flags:    

![/images/2021_08_11_17_25_39_369x454.jpg](/images/2021_08_11_17_25_39_369x454.jpg)

Final layout:    

![/images/2021_08_11_17_26_24_921x217.jpg](/images/2021_08_11_17_26_24_921x217.jpg)

### Write steps
Write via following steps:    

```
➜  ~ cd rpi4 
➜  rpi4 ls
bcm2711-rpi-4-b.dtb  boot  ramdisk.img  system.img  vc4-kms-v3d-pi4.dtbo  vendor.img  zImage
➜  rpi4 sudo dd if=system.img of=/dev/sdb2 bs=1M
768+0 records in
768+0 records out
805306368 bytes (805 MB, 768 MiB) copied, 12.812 s, 62.9 MB/s
➜  rpi4 sudo dd if=vendor.img of=/dev/sdb3 bs=1M
128+0 records in
128+0 records out
134217728 bytes (134 MB, 128 MiB) copied, 2.25087 s, 59.6 MB/s
➜  rpi4 sudo mount /dev/sdb1 /mnt
➜  rpi4 sudo cp boot/* /mnt
➜  rpi4 sudo cp zImage bcm2711-rpi-4-b.dtb ramdisk.img /mnt
➜  rpi4 sudo mkdir /mnt/overlays
➜  rpi4 sudo cp vc4-kms-v3d-pi4.dtbo /mnt/overlays 
➜  rpi4 sudo sync

```
