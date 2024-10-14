+++
title= "lxdonfedora"
date = "2024-10-14T17:16:35+08:00"
description = "lxdonfedora"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install steps:    

```
sudo dnf install snapd
sudo ln -s /var/lib/snapd/snap /snap
snap version
sudo systemctl restart snapd.service
sudo snap install lxd
sudo snap enable lxd
sudo snap services lxd
sudo snap start lxd
```
Add user:    

```
sudo usermod -a -G lxd test
newgrp lxd
```
logout and login again.    
init steps:    

```
lxtest@localhost:~$ lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: 
Do you want to configure a new storage pool? (yes/no) [default=yes]: 
Name of the new storage pool [default=default]: 
Name of the storage backend to use (powerflex, btrfs, ceph, dir, lvm) [default=btrfs]: 
Create a new BTRFS pool? (yes/no) [default=yes]: 
Would you like to use an existing empty block device (e.g. a disk or partition)? (yes/no) [default=no]: 
Size in GiB of the new loop device (1GiB minimum) [default=5GiB]: 20GiB
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to create a new local network bridge? (yes/no) [default=yes]: 
What should the new bridge be called? [default=lxdbr0]: 
What IPv4 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: 
What IPv6 address should be used? (CIDR subnet notation, “auto” or “none”) [default=auto]: none
Would you like the LXD server to be available over the network? (yes/no) [default=no]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes]: 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:
```
show infos:    

```
test@localhost:~$ lxc list
To start your first container, try: lxc launch ubuntu:24.04
Or for a virtual machine: lxc launch ubuntu:24.04 --vm

+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+
test@localhost:~$ lxc network list
+----------+----------+---------+---------------+------+-------------+---------+---------+
|   NAME   |   TYPE   | MANAGED |     IPV4      | IPV6 | DESCRIPTION | USED BY |  STATE  |
+----------+----------+---------+---------------+------+-------------+---------+---------+
| enp6s0u4 | physical | NO      |               |      |             | 0       |         |
+----------+----------+---------+---------------+------+-------------+---------+---------+
| lxdbr0   | bridge   | YES     | 10.12.66.1/24 | none |             | 1       | CREATED |
+----------+----------+---------+---------------+------+-------------+---------+---------+
| virbr0   | bridge   | NO      |               |      |             | 0       |         |
+----------+----------+---------+---------------+------+-------------+---------+---------+

```
search images via:    

```
lxc image alias list images: | grep -i arm64
```
launch instance:    

```
$ lxc launch images:debian/12/arm64 debian-12
Creating debian-12
Retrieving image: metadata: 100% (1.54GB/s)

```
