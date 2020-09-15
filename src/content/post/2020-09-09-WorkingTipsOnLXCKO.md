+++
title= "WorkingTipsOnLXCKO"
date = "2020-09-09T11:18:21+08:00"
description = "WorkingTipsOnLXCKO"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Environment
Ubuntu 18.04.3 LTS, Kernel version:   
`Linux build 5.3.0-62-generic`.   

vagrant box image: `centos76`.   

lxc images:    

```
# apt-get install -y kpartx
# cp ~/.vagrant.d/boxes/centos76/0/libvirt/box.img  /media/sdb/
# cd /media/sdb

root@build:/media/sdb# qemu-img convert box.img box1.img
root@build:/media/sdb# qemu-img info box.img
image: box.img
file format: qcow2
virtual size: 200G (214748364800 bytes)
disk size: 655M
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
root@build:/media/sdb# qemu-img info box1.img
image: box1.img
file format: raw
virtual size: 200G (214748364800 bytes)
disk size: 1.3G

# kpartx -av box1.img 
add map loop2p1 (253:2): 0 419428352 linear 7:2 2048
# mount /dev/mapper/loop2p1 /mnt8/
# ls /mnt8/
bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
# tar -cvzf rootfs.tar.gz -C /mnt8 .
```
Create metadata and import lxc images:    

```
# vim metadata.yaml
architecture: "x86_64"
creation_date: 1599622122 # To get current date in Unix time, use `date +%s` command
properties:
architecture: "x86_64"
description: "CentOS 7.6 for lxc"
os: "redhat"
release: "7.6"
# tar czvf metadata.tar.gz metadata.yaml
# lxc image import metadata.tar.gz rootfs.tar.gz --alias "centos76"
Image imported with fingerprint: 9f53f37e869c643049933dccf8cac9c76107856b1f66955cc2a9d3a55329a060
# lxc image ls
+----------+--------------+--------+-------------+--------+----------+-----------------------------+
|  ALIAS   | FINGERPRINT  | PUBLIC | DESCRIPTION |  ARCH  |   SIZE   |         UPLOAD DATE         |
+----------+--------------+--------+-------------+--------+----------+-----------------------------+
| centos76 | 9f53f37e869c | no     |             | x86_64 | 473.97MB | Sep 9, 2020 at 3:29am (UTC) |
+----------+--------------+--------+-------------+--------+----------+-----------------------------+
```
lxd init using following configuration:    

```
root@build:/media/sdb# lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: 
Name of the storage backend to use (btrfs, dir, lvm) [default=btrfs]: 
Create a new BTRFS pool? (yes/no) [default=yes]: 
Would you like to use an existing block device? (yes/no) [default=no]: ^C
root@build:/media/sdb# lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: 
Name of the storage backend to use (btrfs, dir, lvm) [default=btrfs]: dir
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
Would you like LXD to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 
```
Create a bridge profile:    

```
# lxc profile create bridge
root@build:/media/sdb# cat bridge.profile 
config:
  linux.kernel_modules: ip_tables,ip6_tables,netlink_diag,nf_nat,overlay,br_netfilter
  raw.lxc: "lxc.apparmor.profile=unconfined\nlxc.cap.drop= \nlxc.cgroup.devices.allow=a\nlxc.mount.auto=proc:rw sys:rw"
  security.nesting: "true"
  security.privileged: "true"
description: Bridge LXD profile
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: br0
    type: nic
  root:
    path: /
    pool: default
    type: disk
name: bridge
root@build:/media/sdb# lxc  profile edit bridge < bridge.profile 
root@build:/media/sdb# lxc profile list
+---------+---------+
|  NAME   | USED BY |
+---------+---------+
| bridge  | 0       |
+---------+---------+
| default | 0       |
+---------+---------+
```
Create a instance:    

```
# lxc launch centos76 ko1 --profile bridge
Creating ko1
# lxc ls
+------+---------+-----------------------+------+------------+-----------+
| NAME |  STATE  |         IPV4          | IPV6 |    TYPE    | SNAPSHOTS |
+------+---------+-----------------------+------+------------+-----------+
| ko1  | RUNNING | 10.137.149.190 (eth0) |      | PERSISTENT | 0         |
+------+---------+-----------------------+------+------------+-----------+
```
### How KO Works
Bug fix1: conf/my.cnf mapping.    
Bug fix2: could not running on lxc's docker-compose.    

