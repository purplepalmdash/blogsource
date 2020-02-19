+++
title = "MegaCliForPartition"
date = "2019-03-11T09:12:15+08:00"
description = "MegaCliForPartition"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 目的
分区，以便虚拟化场合.    

硬件:24块硬盘，前两块做成系统盘，其他的则是单独配置:    

![/images/2019_03_11_09_13_41_447x495.jpg](/images/2019_03_11_09_13_41_447x495.jpg)

### 查看现有分区
使用以下命令查看当前分区的情况:    

```
# ./MegaCli64 -PDList -aAll | more
```
注意记录下有关磁盘情形，如:    

![/images/2019_03_11_09_20_11_473x516.jpg](/images/2019_03_11_09_20_11_473x516.jpg)

Slot
Number应该是升序的，0-23是SATA盘，38/39是SAS盘，记录下这一组数值，因为后面我们会针对这些值来分区。Slot Number是0   

查看raid信息:    

```
# ./MegaCli64 -LDInfo -Lall -aAll
```

![/images/2019_03_11_09_47_56_721x783.jpg](/images/2019_03_11_09_47_56_721x783.jpg)

除了virtual driver 0, 其他的都需要被删除。   

貌似是有问题的，先装proxmox再操作。   

### Raid卡配置
F2, 弹出配置:    

![/images/2019_03_11_10_01_08_684x418.jpg](/images/2019_03_11_10_01_08_684x418.jpg)

Delete Drive Group:    

![/images/2019_03_11_10_01_32_466x309.jpg](/images/2019_03_11_10_01_32_466x309.jpg)

最后情况:    

![/images/2019_03_11_10_03_06_597x376.jpg](/images/2019_03_11_10_03_06_597x376.jpg)

### 分区
脚本如下， 4组，而后为热备4个：    

```
./MegaCli64 -CfgLdAdd -r5 [0:0,0:1,0:2,0:3,0:4] WB Direct -a0
./MegaCli64 -CfgLdAdd -r5 [0:5,0:6,0:7,0:8,0:9] WB Direct -a0
./MegaCli64 -CfgLdAdd -r5 [0:10,0:11,0:12,0:13,0:14] WB Direct -a0
./MegaCli64 -CfgLdAdd -r5 [0:15,0:16,0:17,0:18,0:19] WB Direct -a0
./MegaCli64 -PDHSP -Set [-EnclAffinity] [-nonRevertible] -PhysDrv[0:20] -a0
./MegaCli64 -PDHSP -Set [-EnclAffinity] [-nonRevertible] -PhysDrv[0:21] -a0
./MegaCli64 -PDHSP -Set [-EnclAffinity] [-nonRevertible] -PhysDrv[0:22] -a0
./MegaCli64 -PDHSP -Set [-EnclAffinity] [-nonRevertible] -PhysDrv[0:23] -a0
```
