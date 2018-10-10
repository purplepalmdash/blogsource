+++
title = "PromoxSJEnv"
date = "2018-10-10T10:28:26+08:00"
description = "PromoxSJEnv"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Virtual Disk
Virtual CD-ROM:    

![./images/2018_10_10_09_58_53_806x471.jpg](./images/2018_10_10_09_58_53_806x471.jpg)    

### Disk
Storage in Datacenter:    

![/images/2018_10_10_11_13_42_1062x353.jpg](/images/2018_10_10_11_13_42_1062x353.jpg)

Storage on node:    

![/images/2018_10_10_11_14_06_1321x500.jpg](/images/2018_10_10_11_14_06_1321x500.jpg)

Seems only occupy the first `/dev/sda`.    

Remove the exising uneccessary lv and pv:    

```
# lvremove /dev/1kbakvg/1kdf
# vgremove 1kbakvg
```

Add new volume group:     

```
Change from gpt to dos, fdisk /dev/sdb, press o
# vgcreate vms /dev/sdb /dev/sdc /dev/sdd /dev/sde /dev/sdf
# vgdisplay | grep vms
VG Name vms
# lvcreate -L 1024G --thinpool vms_image_pool vms
```

Add LVM-Thin:    

![/images/2018_10_10_11_37_37_507x430.jpg](/images/2018_10_10_11_37_37_507x430.jpg)
Or:   

![/images/2018_10_10_11_47_16_616x234.jpg](/images/2018_10_10_11_47_16_616x234.jpg)

### Upload cdroms
To `/var/lib/vz/template/iso/`    

Create vm:    

![/images/2018_10_10_11_58_20_693x228.jpg](/images/2018_10_10_11_58_20_693x228.jpg)

Choose cd:    

![/images/2018_10_10_11_58_33_708x249.jpg](/images/2018_10_10_11_58_33_708x249.jpg)

Choose Disk Storage size:    

![/images/2018_10_10_11_58_54_616x247.jpg](/images/2018_10_10_11_58_54_616x247.jpg)

Cpus:    

![/images/2018_10_10_11_59_31_627x181.jpg](/images/2018_10_10_11_59_31_627x181.jpg)

Memory:    

![/images/2018_10_10_11_59_45_478x199.jpg](/images/2018_10_10_11_59_45_478x199.jpg)

Network:    

![/images/2018_10_10_12_00_06_538x286.jpg](/images/2018_10_10_12_00_06_538x286.jpg)

Then start the machine, begin install:    

![/images/2018_10_10_12_01_10_745x572.jpg](/images/2018_10_10_12_01_10_745x572.jpg)

### vms
Start on boot
![/images/2018_10_10_14_20_52_767x424.jpg](/images/2018_10_10_14_20_52_767x424.jpg)

