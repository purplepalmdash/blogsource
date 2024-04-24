+++
title= "InstallDockerOnPiKVM"
date = "2024-04-24T09:17:53+08:00"
description = "InstallDockerOnPiKVM"
keywords = ["Technology"]
categories = ["Technology"]
+++
Remount `/` for rw mode:    

```
mount -o rw,remount -t ext4  /
```
Configure repository for using ustc repository:    

```
# cat /etc/pacman.d/mirrorlist 
Server = https://mirrors.ustc.edu.cn/archlinuxarm/$arch/$repo
# pacman -Sy
# pacman -S docker nfs-utils
```
Change docker storage location:     

```
# mount -t nfs 192.168.1.8:/media/sda /media/nfs
# mkdir -p /media/nfs/docker
# vim /etc/docker/daemon.json
{
    "data-root": "/media/nfs/docker"
}
# systemctl start docker
```
Combine all of the bash scripts:    

```
# cat startdocker.sh 
#!/bin/bash
mount -o rw,remount -t ext4  /
mount -t nfs 192.168.1.8:/media/sda /media/nfs
systemctl start docker
```
