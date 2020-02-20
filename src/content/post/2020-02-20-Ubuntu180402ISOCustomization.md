+++
title = "Customize Ubuntu 18.04.2 iso"
date = "2020-02-20T21:52:53+08:00"
description = "CustomizeUbuntuISO"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Material
Ubuntu18.04.2 amd64 installation iso, md5sum is listed as:    

```
34416ff83179728d54583bf3f18d42d2  ubuntu-18.04.2-server-amd64.iso
```
### Steps
Using poweriso open this iso file:    

![/images/2020_02_20_21_54_17_797x300.jpg](/images/2020_02_20_21_54_17_797x300.jpg)

preseed directory now:    

![/images/2020_02_20_21_54_42_581x201.jpg](/images/2020_02_20_21_54_42_581x201.jpg)

replace the txt.cfg file:    

![/images/2020_02_20_21_58_54_596x431.jpg](/images/2020_02_20_21_58_54_596x431.jpg)

For uefi mode, just replace the boot/grub configuration file:    

![/images/2020_02_20_22_00_30_617x448.jpg](/images/2020_02_20_22_00_30_617x448.jpg)

Save as then we could get a new iso.   
