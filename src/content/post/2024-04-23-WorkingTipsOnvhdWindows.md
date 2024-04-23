+++
title= "WorkingTipsOnvhdWindows"
date = "2024-04-23T19:20:58+08:00"
description = "WorkingTipsOnvhdWindows"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. basic disk preparation
Disk layout:    

![/images/2024_04_23_19_21_10_808x560.jpg](/images/2024_04_23_19_21_10_808x560.jpg)

Install disk :   

![/images/2024_04_23_19_21_47_713x325.jpg](/images/2024_04_23_19_21_47_713x325.jpg)

Choose efi and ext4 partition:    

![/images/2024_04_23_19_22_31_841x584.jpg](/images/2024_04_23_19_22_31_841x584.jpg)

edit visudo , install vim and opensshserver, update, then save this disk.  

```
sudo virsh undefine 0000_ventoyvhd
qemu-img create -f qcow2 -b ventoyvhd.qcow2 -F qcow2 0000_win10vhd.qcow2
qemu-img create -f qcow2 -b ventoyvhd.qcow2 -F qcow2 0001_win11vhd.qcow2
``` 
### 2. win11 installed on vhd
Select `Professional`:   

![/images/2024_04_23_19_36_27_624x486.jpg](/images/2024_04_23_19_36_27_624x486.jpg)

Select `customized: ...  `

![/images/2024_04_23_19_37_31_757x483.jpg](/images/2024_04_23_19_37_31_757x483.jpg)

Create vdisk using following commands:    

![/images/2024_04_23_19_39_52_786x514.jpg](/images/2024_04_23_19_39_52_786x514.jpg)

Refresh the disk(before):   

![/images/2024_04_23_19_40_11_596x458.jpg](/images/2024_04_23_19_40_11_596x458.jpg)

After refreshment:    

![/images/2024_04_23_19_40_30_625x472.jpg](/images/2024_04_23_19_40_30_625x472.jpg)

Select driver 1(80GB):    

![/images/2024_04_23_19_40_50_653x482.jpg](/images/2024_04_23_19_40_50_653x482.jpg)

Begin installation:   

![/images/2024_04_23_19_41_08_411x276.jpg](/images/2024_04_23_19_41_08_411x276.jpg)

启动，蓝屏。
### 3. win10 installed on vhd
Create the disk vhd name `win10.vhd`:    

![/images/2024_04_23_19_51_49_625x482.jpg](/images/2024_04_23_19_51_49_625x482.jpg)



