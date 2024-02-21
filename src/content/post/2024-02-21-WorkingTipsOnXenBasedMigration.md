+++
title= "WorkingTipsOnXenBasedMigration"
date = "2024-02-21T11:02:49+08:00"
description = "WorkingTipsOnXenBasedMigration"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. 安装Xenserver 
创建启动盘:    

```
$ sudo dd if=./XenServer8_2024-01-09.iso of=/dev/sdb bs=1M && sudo sync
```
U盘启动工作站开始安装:    

![/images/2024_02_21_11_05_26_963x436.jpg](/images/2024_02_21_11_05_26_963x436.jpg)

![/images/2024_02_21_11_06_10_798x581.jpg](/images/2024_02_21_11_06_10_798x581.jpg)

![/images/2024_02_21_11_06_23_1226x540.jpg](/images/2024_02_21_11_06_23_1226x540.jpg)

![/images/2024_02_21_11_06_34_1233x970.jpg](/images/2024_02_21_11_06_34_1233x970.jpg)

![/images/2024_02_21_11_06_47_1163x512.jpg](/images/2024_02_21_11_06_47_1163x512.jpg)

![/images/2024_02_21_11_07_11_1327x558.jpg](/images/2024_02_21_11_07_11_1327x558.jpg)

![/images/2024_02_21_11_07_25_936x470.jpg](/images/2024_02_21_11_07_25_936x470.jpg)

![/images/2024_02_21_11_07_39_754x388.jpg](/images/2024_02_21_11_07_39_754x388.jpg)

![/images/2024_02_21_11_07_55_879x508.jpg](/images/2024_02_21_11_07_55_879x508.jpg)

![/images/2024_02_21_11_09_07_902x448.jpg](/images/2024_02_21_11_09_07_902x448.jpg)

![/images/2024_02_21_11_09_23_729x602.jpg](/images/2024_02_21_11_09_23_729x602.jpg)

![/images/2024_02_21_11_09_38_811x587.jpg](/images/2024_02_21_11_09_38_811x587.jpg)

![/images/2024_02_21_11_09_51_813x421.jpg](/images/2024_02_21_11_09_51_813x421.jpg)

![/images/2024_02_21_11_10_03_967x421.jpg](/images/2024_02_21_11_10_03_967x421.jpg)

![/images/2024_02_21_11_10_16_1199x236.jpg](/images/2024_02_21_11_10_16_1199x236.jpg)

![/images/2024_02_21_11_11_55_1009x292.jpg](/images/2024_02_21_11_11_55_1009x292.jpg)

![/images/2024_02_21_11_15_43_1003x458.jpg](/images/2024_02_21_11_15_43_1003x458.jpg)


登陆:    

```
[root@xenserver-tyy ~]# cat /etc/issue
XenServer 8

System Booted: 2024-02-21 11:16

Your XenServer host has now finished booting. 
To manage this server please use the XenCenter application. 
You can install XenCenter for Windows from https://www.xenserver.com/downloads.

You can connect to this system using one of the following network
addresses:

IP address not configured

[root@xenserver-tyy ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1706         110        1278           8         318        1534
Swap:          1023           0        1023
[root@xenserver-tyy ~]# 
```
### XenCenter

![/images/2024_02_21_11_20_15_538x406.jpg](/images/2024_02_21_11_20_15_538x406.jpg)

![/images/2024_02_21_11_20_25_613x458.jpg](/images/2024_02_21_11_20_25_613x458.jpg)

![/images/2024_02_21_11_20_36_489x383.jpg](/images/2024_02_21_11_20_36_489x383.jpg)

![/images/2024_02_21_11_20_50_505x385.jpg](/images/2024_02_21_11_20_50_505x385.jpg)

![/images/2024_02_21_11_21_12_1006x755.jpg](/images/2024_02_21_11_21_12_1006x755.jpg)

增加新xenserver:    

![/images/2024_02_21_11_21_35_591x305.jpg](/images/2024_02_21_11_21_35_591x305.jpg)

![/images/2024_02_21_11_21_52_652x291.jpg](/images/2024_02_21_11_21_52_652x291.jpg)

添加的`xenserver-tyy`：    

![/images/2024_02_21_11_22_10_989x712.jpg](/images/2024_02_21_11_22_10_989x712.jpg)

import类型只支持:    

![/images/2024_02_21_11_32_27_429x125.jpg](/images/2024_02_21_11_32_27_429x125.jpg)

所以需要转换为vmdk?    

![/images/2024_02_21_12_11_30_812x534.jpg](/images/2024_02_21_12_11_30_812x534.jpg)

![/images/2024_02_21_12_11_51_810x533.jpg](/images/2024_02_21_12_11_51_810x533.jpg)

![/images/2024_02_21_12_12_03_822x429.jpg](/images/2024_02_21_12_12_03_822x429.jpg)

![/images/2024_02_21_12_18_35_810x551.jpg](/images/2024_02_21_12_18_35_810x551.jpg)

### 转换格式

