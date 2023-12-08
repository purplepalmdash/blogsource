+++
title= "WorkingTipsOnDeployment"
date = "2023-12-05T14:05:38+08:00"
description = "WorkingTipsOnDeployment"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. Partition Preparation
Shrink the home partition:    

```
tar -czvf /root/home.tgz -C /home .
tar -tvf /root/home.tgz
umount /dev/mapper/centos-home
lvremove /dev/mapper/centos-home
lvcreate -L 40GB -n home centos
mkfs.xfs /dev/centos/home
mount /dev/mapper/centos-home
lvextend -r -l +100%FREE /dev/mapper/centos-root
tar -xzvf /root/home.tgz -C /home
```
Create the gpt partition on nvme disk:    

```
gdisk /dev/nvme0n1
gdisk /dev/nvme1n1
gdisk /dev/nvme2n1
gdisk /dev/nvme3n1

o Enter for new empty GUID partition table (GPT)
y Enter to confirm your decision
n Enter for new partition
Enter for default of first partition
Enter for default of the first sector
Enter for default of the last sector
fd00 Enter for Linux RAID type
w Enter to write changes
y Enter to confirm your decision
```
Create the raid1 using mdadm:     

```
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/nvme0n1p1 /dev/nvme1n1p1
mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/nvme2n1p1 /dev/nvme3n1p1
```
Examine the partition via:      

```
lsblk

......
nvme2n1         259:2    0     7T  0 disk  
└─nvme2n1p1     259:5    0     7T  0 part  
  └─md1           9:1    0     7T  0 raid1 
nvme1n1         259:1    0     7T  0 disk  
└─nvme1n1p1     259:4    0     7T  0 part  
  └─md0           9:0    0     7T  0 raid1 
......
```
Create the pv:     

```
pvcreate /dev/md0
pvcreate /dev/md1
```
Create the vg:    

```
# vgcreate vmvolume /dev/md0
  Volume group "vmvolume" successfully created
# vgextend vmvolume /dev/md1
  Volume group "vmvolume" successfully extended
```
###  2. Create the 
