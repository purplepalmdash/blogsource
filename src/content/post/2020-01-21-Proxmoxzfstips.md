+++
title = "Promoxzfstips"
date = "2020-01-21T16:18:04+08:00"
description = "Promoxzfstips"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Steps
之前:    

![/images/2020_01_21_16_18_33_1171x255.jpg](/images/2020_01_21_16_18_33_1171x255.jpg)

ssh登录:    

`./MegaCli64 -LDInfo -LALL -aAll`查看VD信息， 其中VD0（600G）不需要动.     

![/images/2020_01_21_16_20_59_871x328.jpg](/images/2020_01_21_16_20_59_871x328.jpg)

`./MegaCli64 -PDList -aAll | grep -i adapter`得到adapter数值:    

![/images/2020_01_21_16_22_31_783x47.jpg](/images/2020_01_21_16_22_31_783x47.jpg)


删除VD1-VD3:    

```
# ./MegaCli64 -cfglddel -L1 -a0
# ./MegaCli64 -cfglddel -L2 -a0
# ./MegaCli64 -cfglddel -L3 -a0
```
当前VD：    

![/images/2020_01_21_16_23_26_850x391.jpg](/images/2020_01_21_16_23_26_850x391.jpg)

查看PD对应磁盘:    

```
# ./MegaCli64 -PDList -aAll | more
两个600G的是0和1, 其他的随便动
```
查看多少块盘:    

```
# ./MegaCli64 -PDList -aAll |  grep 'Slot Number'
```
![/images/2020_01_21_16_25_57_813x589.jpg](/images/2020_01_21_16_25_57_813x589.jpg)

这里注意,`2,3` 是没有，从`4~27`为slot number.   

得到Enclosure ID:    

```
# ./MegaCli64 -PDList -aAll | grep 'Enclosure'
为9
```

开始做24个raid0:     

```
# ./MegaCli64 -CfgLdAdd -r0 [9:4] -a0
# ./MegaCli64 -CfgLdAdd -r0 [9:5] -a0
......
# ./MegaCli64 -CfgLdAdd -r0 [9:26] -a0
# ./MegaCli64 -CfgLdAdd -r0 [9:27] -a0
```
脚本:    

![/images/2020_01_21_16_30_40_450x557.jpg](/images/2020_01_21_16_30_40_450x557.jpg)

`lsblk`查看磁盘信息:    

![/images/2020_01_21_16_31_31_562x891.jpg](/images/2020_01_21_16_31_31_562x891.jpg)

删除多余分区, sdb/sdn/sdr:    

![/images/2020_01_21_16_32_53_643x819.jpg](/images/2020_01_21_16_32_53_643x819.jpg)


### add zfs pool
命令行下添加:     

```
# zpool create -f -o ashift=12 vmpool raidz2 /dev/sdb /dev/sdc /dev/sdd /dev/sde /dev/sdf /dev/sdg /dev/sdh /dev/sdi
# zpool add -f -o ashift=12 vmpool raidz2 /dev/sdj /dev/sdk /dev/sdl /dev/sdm /dev/sdn /dev/sdo /dev/sdp /dev/sdq
# zpool add -f -o ashift=12 vmpool raidz2 /dev/sdr /dev/sds /dev/sdt /dev/sdu /dev/sdv /dev/sdw /dev/sdx /dev/sdy
# zpool list
NAME     SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
vmpool   130T  1.97M   130T         -     0%     0%  1.00x  ONLINE  -
# zfs list
NAME     USED  AVAIL  REFER  MOUNTPOINT
vmpool   819K  89.9T   205K  /vmpool
```

Add in proxmox:   

![/images/2020_01_21_16_34_13_609x424.jpg](/images/2020_01_21_16_34_13_609x424.jpg)

设置参数:    

![/images/2020_01_21_16_34_50_619x228.jpg](/images/2020_01_21_16_34_50_619x228.jpg)

可用：    

![/images/2020_01_21_16_35_18_891x159.jpg](/images/2020_01_21_16_35_18_891x159.jpg)

使用方法:    

![/images/2020_01_21_16_37_23_712x533.jpg](/images/2020_01_21_16_37_23_712x533.jpg)    

