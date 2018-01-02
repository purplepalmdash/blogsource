+++
date = "2017-05-24T14:32:50+08:00"
categories = ["Virtualization"]
keywords = ["migration"]
description = "migration for centos 7"
title = "MigrationForCentOS7"

+++
### 目的
使用磁盘克隆的方式快速安装、部署系统。    

### 验证环境
Virt-manager, CentOS 7 ISO安装盘

### 安装注意事项
分区时选择xfs(CentOS默认)， 选择LVM分区。    

![/images/2017_05_24_15_16_47_460x471.jpg](/images/2017_05_24_15_16_47_460x471.jpg)

The disk partition should be `Automatically create them`.    

![/images/2017_05_24_15_17_58_595x456.jpg](/images/2017_05_24_15_17_58_595x456.jpg)

安装时选择minimum installation. 安装完毕后，关闭虚拟机
### 安装后调整
复制安装好的硬盘:    

```
$ ls
CentOS5G.qcow2
$ cp CentOS5G.qcow2 Duplicated.qcow2
$ ls
CentOS5G.qcow2  Duplicated.qcow2
```
创建一个中转盘，一个大小为50G的目标盘,
中转盘用于存放克隆文件，而目标盘则是我们将克隆文件拷贝过去的盘。    

```
$ qemu-img create -f qcow2 Middle.qcow2 16G
Formatting 'Middle.qcow2', fmt=qcow2 size=17179869184 encryption=off cluster_size=65536 lazy_refcounts=off refcount_bits=16
$ qemu-img create -f qcow2 Dest.qcow2 50G
Formatting 'Dest.qcow2', fmt=qcow2 size=53687091200 encryption=off cluster_size=65536 lazy_refcounts=off refcount_bits=16
```
在virt-manager中，依次添加剩余的三块硬盘.    

![/images/2017_05_24_14_49_04_614x394.jpg](/images/2017_05_24_14_49_04_614x394.jpg)

查看磁盘格局为,
可以看到，vdb和vda其实一样，但因为vdb没有被挂载，所以可以直接从其上往目标盘使用dd进行拷贝:    

```
# fdisk -l

Disk /dev/vda: 8589 MB, 8589934592 bytes, 16777216 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000c7c16

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048     1026047      512000   83  Linux
/dev/vda2         1026048    16777215     7875584   8e  Linux LVM

Disk /dev/vdb: 8589 MB, 8589934592 bytes, 16777216 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000c7c16

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1   *        2048     1026047      512000   83  Linux
/dev/vdb2         1026048    16777215     7875584   8e  Linux LVM

Disk /dev/vdc: 17.2 GB, 17179869184 bytes, 33554432 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/vdd: 53.7 GB, 53687091200 bytes, 104857600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

中转盘可以用于制作img文件,
这里我们先不制作img文件，我们直接从复制的盘往目标盘写入:    

```
[root@localhost ~]# dd if=/dev/vdb of=/dev/vdd bs=1M
8192+0 records in
8192+0 records out
8589934592 bytes (8.6 GB) copied, 150.443 s, 57.1 MB/s
```
现在查看`/dev/vdd`的磁盘结构，可以看到与`/dev/vda`和`/dev/vdb`是一样的.    

扩展`/dev/vdd`的分区，从8G 扩展到50G：    

```
# fdisk /dev/vdd
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): d
Partition number (1,2, default 2): 
Partition 2 is deleted

Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (2-4, default 2): 
First sector (1026048-104857599, default 1026048): 
Using default value 1026048
Last sector, +sectors or +size{K,M,G} (1026048-104857599, default 104857599): 
Using default value 104857599
Partition 2 of type Linux and of size 49.5 GiB is set

Command (m for help): 
Command (m for help): p

Disk /dev/vdd: 53.7 GB, 53687091200 bytes, 104857600 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000c7c16

   Device Boot      Start         End      Blocks   Id  System
/dev/vdd1   *        2048     1026047      512000   83  Linux
/dev/vdd2         1026048   104857599    51915776   83  Linux

Command (m for help): t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
现在关闭虚拟机，使用`Dest.qcow2`文件创建一台新的虚拟机，    

![/images/2017_05_24_15_00_08_582x284.jpg](/images/2017_05_24_15_00_08_582x284.jpg)

### 扩展分区
进入系统后，看到磁盘分区依然保持为8G下的磁盘格局    

```
# df -HT
Filesystem                Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos00-root xfs       7.2G  897M  6.3G  13% /
devtmpfs                  devtmpfs  510M     0  510M   0% /dev
tmpfs                     tmpfs     521M     0  521M   0% /dev/shm
tmpfs                     tmpfs     521M  7.0M  514M   2% /run
tmpfs                     tmpfs     521M     0  521M   0% /sys/fs/cgroup
/dev/vda1                 xfs       521M  131M  391M  26% /boot
tmpfs                     tmpfs     105M     0  105M   0% /run/user/0
```
使用以下命令，扩展pv(physical volumn):    

```
[root@localhost ~]# pvs
  PV         VG       Fmt  Attr PSize PFree 
  /dev/vda2  centos00 lvm2 a--  7.51g 40.00m
[root@localhost ~]# pvresize /dev/vda2
  Physical volume "/dev/vda2" changed
  1 physical volume(s) resized / 0 physical volume(s) not resized
[root@localhost ~]# pvs
  PV         VG       Fmt  Attr PSize  PFree 
  /dev/vda2  centos00 lvm2 a--  49.51g 42.04g
```

扩展根分区, 注意mapper的名称:    

```
[root@localhost ~]# vgs
  VG       #PV #LV #SN Attr   VSize  VFree 
  centos00   1   2   0 wz--n- 49.51g 42.04g
[root@localhost ~]# ls /dev/centos00/
root  swap
[root@localhost ~]# lvextend -l +100%FREE /dev/centos00/root 
  Size of logical volume centos00/root changed from 6.67 GiB (1707 extents) to 48.71 GiB (12469 extents).
  Logical volume root successfully resized.
```
扩展xfs:    

```
[root@localhost ~]# xfs_growfs /dev/centos00/root
meta-data=/dev/mapper/centos00-root isize=256    agcount=4, agsize=436992 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0        finobt=0
data     =                       bsize=4096   blocks=1747968, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 1747968 to 12768256
[root@localhost ~]# df -HT
Filesystem                Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos00-root xfs        53G  897M   52G   2% /
devtmpfs                  devtmpfs  510M     0  510M   0% /dev
tmpfs                     tmpfs     521M     0  521M   0% /dev/shm
tmpfs                     tmpfs     521M  7.0M  514M   2% /run
tmpfs                     tmpfs     521M     0  521M   0% /sys/fs/cgroup
/dev/vda1                 xfs       521M  131M  391M  26% /boot
tmpfs                     tmpfs     105M     0  105M   0% /run/user/0
```
现在重新启动系统，发现根分区已经扩展到了50G的空间。    
