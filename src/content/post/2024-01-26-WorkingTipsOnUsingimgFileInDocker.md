+++
title= "WorkingTipsOnUsingimgFileInDocker"
date = "2024-01-26T09:58:15+08:00"
description = "WorkingTipsOnUsingimgFileInDocker"
keywords = ["Technology"]
categories = ["Technology"]
+++
准备Img文件:     

```
$ wget https://mirrors.ustc.edu.cn/raspberry-pi-os-images/raspios_lite_arm64/images/raspios_lite_arm64-2023-12-11/2023-12-11-raspios-bookworm-arm64-lite.img.xz
$ unxz 2023-12-11-raspios-bookworm-arm64-lite.img.xz 
$ ls
2023-12-11-raspios-bookworm-arm64-lite.img
```
以准备好的img文件设置loop挂载点, 而后检查挂载点:     

```
$ sudo losetup -fP --show  2023-12-11-raspios-bookworm-arm64-lite.img
/dev/loop17
$ sudo fdisk -l /dev/loop17
Disk /dev/loop17: 2.55 GiB, 2738880512 bytes, 5349376 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4e639091

Device        Boot   Start     End Sectors  Size Id Type
/dev/loop17p1         8192 1056767 1048576  512M  c W95 FAT32 (LBA)
/dev/loop17p2      1056768 5349375 4292608    2G 83 Linux
```
将`/dev/loop17p2`分区挂载到/mnt9后，而后透传给容器:     

```
$  sudo mount /dev/loop17p2 /mnt9/
$ sudo docker run -it -v /mnt9:/raspbian rockylinux:9 bash
```
在容器中检查`/raspbian`挂载点所在的位置，分区大小为2G:     

```
[root@34e78094dace /]# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay         1.8T  1.5T  259G  86% /
tmpfs            64M     0   64M   0% /dev
shm              64M     0   64M   0% /dev/shm
/dev/loop17p2   2.0G  1.6G  330M  83% /raspbian
/dev/nvme0n1p2  1.8T  1.5T  259G  86% /etc/hosts
tmpfs            63G     0   63G   0% /proc/asound
tmpfs            63G     0   63G   0% /proc/acpi
tmpfs            63G     0   63G   0% /proc/scsi
tmpfs            63G     0   63G   0% /sys/firmware
```
