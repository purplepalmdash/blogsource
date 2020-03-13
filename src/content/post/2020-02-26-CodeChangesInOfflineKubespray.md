+++
title= "CodeChangesInOfflineKubespray"
date = "2020-02-26T20:17:37+08:00"
description = "CodeChangesInOfflineKubespray"
keywords = ["Technology"]
categories = ["Technology"]
+++
For setting listor storage on kubernetes offlinely.   

### Linstor Package Preparation
Fetch the deb pkgs in docker for offline usage:     

```
add-apt-repository ppa:linbit/linbit-drbd9-stack
apt-get update

apt install -y drbd-dkms drbd-utils
```
Transfer the packages onto the deploy node and update the repository, then install in all of the nodes via:     

```
# apt-get update -y && DEBIAN_FRONTEND=noninteractive apt-get install -y drbd-dkms drbd-utils 2>&1 
```
`noninteractive` make sure the postfix in default configuration.   
### Storage Preparation
Create qcow2 files for vm usage:    

```
# qemu-img create -f qcow2 /media/sda/listor1.qcow2 500G
# qemu-img create -f qcow2 /media/sdb/listor1.qcow2 500G
# qemu-img create -f qcow2 /media/sdc3/listor1.qcow2 500G
```
Attach them to vms:    

![/images/2020_02_29_11_00_05_541x322.jpg](/images/2020_02_29_11_00_05_541x322.jpg)

Each of the node have the vdb installed:    

```
# fdisk -l
Disk /dev/vdb: 500 GiB, 536870912000 bytes, 1048576000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

Create pv, vg, lv via following commands:    

```
# pvcreate /dev/vdb && vgcreate vg /dev/vdb && lvcreate -l 100%FREE --thinpool vg/lvmthinpool
```
