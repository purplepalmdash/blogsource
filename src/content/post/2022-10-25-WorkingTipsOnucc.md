+++
title= "WorkingTipsOnucc"
date = "2022-10-25T09:48:49+08:00"
description = "WorkingTipsOnucc"
keywords = ["Technology"]
categories = ["Technology"]
+++
Create a inner networking for testing:   

![/images/2022_10_25_09_48_58_519x557.jpg](/images/2022_10_25_09_48_58_519x557.jpg)

Select the inner networking for imported vm:    

![/images/2022_10_25_09_49_48_719x410.jpg](/images/2022_10_25_09_49_48_719x410.jpg)

Change vm's ip address configuration:    

![/images/2022_10_25_09_52_46_481x379.jpg](/images/2022_10_25_09_52_46_481x379.jpg)

Check the changed ip configuration:   

![/images/2022_10_25_09_53_37_639x193.jpg](/images/2022_10_25_09_53_37_639x193.jpg)

visit `https://10.17.18.2` for changing ucc server configuration:    

![/images/2022_10_25_09_54_54_541x562.jpg](/images/2022_10_25_09_54_54_541x562.jpg)

Changed the `server address`:    

![/images/2022_10_25_09_59_20_312x164.jpg](/images/2022_10_25_09_59_20_312x164.jpg)

Change the dhcp service:    

![/images/2022_10_25_10_01_51_448x397.jpg](/images/2022_10_25_10_01_51_448x397.jpg)

Delete all of the registed server:    

![/images/2022_10_25_10_04_49_817x322.jpg](/images/2022_10_25_10_04_49_817x322.jpg)
### Verification
Create 2 vm images file under `/var/lib/libvirt/images`:    

```
# qemu-img create -f qcow2 win1.qcow2 50G
# qemu-img create -f qcow2 win2.qcow2 50G
```
Manually import :    

![/images/2022_10_25_10_10_19_423x263.jpg](/images/2022_10_25_10_10_19_423x263.jpg)

2 core, 4096 MiB memory. Select win1.qcow2:    

![/images/2022_10_25_10_11_07_396x234.jpg](/images/2022_10_25_10_11_07_396x234.jpg)

uefi relatd configuration:    

![/images/2022_10_25_10_13_03_786x544.jpg](/images/2022_10_25_10_13_03_786x544.jpg)

Select Nic:    

![/images/2022_10_25_10_13_28_581x324.jpg](/images/2022_10_25_10_13_28_581x324.jpg)

Change the boot options:    

![/images/2022_10_25_10_15_40_469x345.jpg](/images/2022_10_25_10_15_40_469x345.jpg)

Configuration:   

![/images/2022_10_25_10_20_34_465x219.jpg](/images/2022_10_25_10_20_34_465x219.jpg)

Initialize the disk:    

![/images/2022_10_25_10_21_09_1830x619.jpg](/images/2022_10_25_10_21_09_1830x619.jpg)

Succeed:   

![/images/2022_10_25_10_22_25_1172x265.jpg](/images/2022_10_25_10_22_25_1172x265.jpg)

Login with `user1`:    

![/images/2022_10_25_10_23_12_448x582.jpg](/images/2022_10_25_10_23_12_448x582.jpg)

Select the `TCI` for downloading:    

![/images/2022_10_25_10_23_46_541x315.jpg](/images/2022_10_25_10_23_46_541x315.jpg)



### Virtualbox way
Create vm and import:     

![/images/2022_10_25_11_28_49_964x545.jpg](/images/2022_10_25_11_28_49_964x545.jpg)

Import vm disk:    

![/images/2022_10_25_11_30_03_1059x541.jpg](/images/2022_10_25_11_30_03_1059x541.jpg)

Create network(without dhcp):    

![/images/2022_10_25_11_31_46_520x218.jpg](/images/2022_10_25_11_31_46_520x218.jpg)

Choose the network:   

![/images/2022_10_25_11_32_23_802x439.jpg](/images/2022_10_25_11_32_23_802x439.jpg)

