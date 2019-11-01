+++
title = "WorkingtipsOnZFSOnProxmox"
date = "2019-11-01T09:54:43+08:00"
description = "WorkingtipsOnZFSOnProxmox"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Environment
2 disks(sas) raid1 as the system partition.    
24 disks(SATA), each disk has 2 TB.     
CPU: Intel Xeon CPU e5-2650 v3 @ 2.30GHZ.     
256G memory.     
### Disk configuration
use MegaRAID/MegaCli for configurating the disk parameters.     

Get the parameters:    

```
# ./Megacli64 -LDInfo -LALL -aAll
Virtual Drive: 24 (Target Id: 24)
Name                :
RAID Level          : Primary-0, Secondary-0, RAID Level Qualifier-0
Size                : 1.817 TB
State               : Optimal
Strip Size          : 256 KB
Number Of Drives    : 1
Span Depth          : 1
Default Cache Policy: WriteBack, ReadAhead, Direct, No Write Cache if Bad BBU
Current Cache Policy: WriteThrough, ReadAhead, Direct, No Write Cache if Bad BBU
Access Policy       : Read/Write
Disk Cache Policy   : Disk's Default
Encryption Type     : None

Exit Code: 0x00
``` 
Notice we have the `ReadAhead` in `Current Cache Policy`, we need to turn off this parameter in order to let zfs runs fast.     

```
# ./MegaCli64 -LDSetProp -NORA -Immediate -Lall -aAll
.....
Current Cache Policy: WriteThrough, ReadAheadNone, Direct, No Write Cache if Bad BBU
.....
```
Create the raidz2 vmpool via following commands:     

```
# zpool create -f -o ashift=12 vmpool raidz2 /dev/sdb /dev/sdc /dev/sdd /dev/sde /dev/sdf /dev/sdg /dev/sdh /dev/sdi
# zpool add -f -o ashift=12 vmpool raidz2 /dev/sdj ~ /dev/sdq
# zpool add -f -o ashift=12 vmpool raidz2 /dev/sdr ~ /dev/sdy
```
### zpool info
Get the information of zfs via following:     

```
# zpool list
NAME     SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
vmpool  43.5T   132G  43.4T         -     0%     0%  1.00x  ONLINE  -
# zfs list
NAME                     USED  AVAIL  REFER  MOUNTPOINT
vmpool                  93.6G  29.9T   205K  /vmpool
vmpool/base-100-disk-1  8.17G  29.9T  8.17G  -
vmpool/vm-101-disk-1    45.3G  29.9T  45.3G  -
vmpool/vm-102-disk-1    40.1G  29.9T  40.1G  -
```
Adding the zfs pool into the proxmox on GUI, ignore the steps because it's too simple. 
