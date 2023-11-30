+++
title= "TipsOnIDVVDIFusion"
date = "2023-11-29T15:05:19+08:00"
description = "TipsOnIDVVDIFusion"
keywords = ["Technology"]
categories = ["Technology"]
+++
Modification steps:    

### 1. Qemu modification
Rebuild qemu:       

```
sudo apt install -y librbd-dev
cd qemu-7.1.0/
./configure --target-list=x86_64-softmmu --enable-debug --disable-docs --disable-virglrenderer --prefix=/usr --enable-virtfs --enable-libusb --disable-debug-tcg --audio-drv-list=pa,alsa --enable-spice --enable-rbd
make -j8 && make install
```

### 2. Ceph Related
Install `ceph-common`:    

```
$ apt-cache policy ceph-common
ceph-common:
  Installed: (none)
  Candidate: 17.2.6-0ubuntu0.22.04.2
  Version table:
     17.2.6-0ubuntu0.22.04.2 500
        500 http://mirrors.ustc.edu.cn/ubuntu jammy-updates/main amd64 Packages
     17.2.5-0ubuntu0.22.04.3 500
        500 http://mirrors.ustc.edu.cn/ubuntu jammy-security/main amd64 Packages
     17.1.0-0ubuntu3 500
        500 http://mirrors.ustc.edu.cn/ubuntu jammy/main amd64 Packages
$ sudo apt install -y ceph-common
```
Define virsh's secret:    

```
$ cat secret.txt 
<secret ephemeral='no' private='no'>
  <uuid>xxxxxxxxxxxxxxxxxxxxxxxx</uuid>
  <usage type='ceph'>
    <name>client.cinder secret</name>
  </usage>
</secret>
$ virsh secret-define secret.txt
Secret xxxxxxxxxxxxxxxxxx created
```
Set the secret:    

```
# virsh 
Welcome to virsh, the virtualization interactive terminal.

Type:  'help' for help with commands
       'quit' to quit

virsh # secret-set-value de12b241-6087-47e1-9d4f-c8baf5ff4968 aofuowguoewogowaugowogwe
error: Passing secret value as command-line argument is insecure!
Secret value set

virsh # secret-get-value de12b241-6087-47e1-9d4f-c8baf5ff4968
aofuowguoewogowaugowogwe
```
Get the rbd for the vdi instance:    

```
$ sudo virsh dumpxml privatedefaulttenant-default_ebc6fef5-3447-4788-8980-f780ad336399 | grep rbd
      <source protocol='rbd' name='ceph-vm-pool-1/volume-d1cb2b42-fe78-41ef-beb3-a6fc12d6e761'>
```
Get the info in local machine via `rbd` command:     

```
# rbd --id cinder info ceph-vm-pool-1/volume-d1cb2b42-fe78-41ef-beb3-a6fc12d6e761
2023-11-30T10:36:43.676+0800 7f0e170e64c0 -1 asok(0x55b5039f6090) AdminSocketConfigObs::init: failed: AdminSocket::bind_and_listen: failed to bind the UNIX domain socket to '/var/run/ceph/guests/ceph-client.cinder.3827.94235938232352.asok': (2) No such file or directory
rbd image 'volume-d1cb2b42-fe78-41ef-beb3-a6fc12d6e761':
	size 80 GiB in 20480 objects
	order 22 (4 MiB objects)
	snapshot_count: 0
	id: 827dd9323d6248
	block_name_prefix: rbd_data.827dd9323d6248
	format: 2
	features: layering, exclusive-lock, object-map, fast-diff, deep-flatten, operations
	op_features: clone-child
	flags: 
	create_timestamp: Wed Nov  8 10:09:37 2023
	access_timestamp: Thu Nov 30 10:35:31 2023
	modify_timestamp: Thu Nov 30 10:36:33 2023
	parent: ceph-vm-pool-1/volume-d8a795bd-48c4-425e-82ba-a22a244778ad@snap-ed0919b6-a4a7-4e6d-a447-dbe89f27bbb8
	overlap: 80 GiB
```
Mount the remote rbd to local:    

```
# rbd --id cinder map ceph-vm-pool-1/volume-d1cb2b42-fe78-41ef-beb3-a6fc12d6e761 -p testpool
2023-11-30T10:37:26.447+0800 7fcdaaef84c0 -1 asok(0x5576928dc090) AdminSocketConfigObs::init: failed: AdminSocket::bind_and_listen: failed to bind the UNIX domain socket to '/var/run/ceph/guests/ceph-client.cinder.3846.93967753279520.asok': (2) No such file or directory
/dev/rbd0
# lsblk | grep rbd0
rbd0        252:0    0    80G  0 disk 
├─rbd0p1    252:1    0   500M  0 part 
└─rbd0p2    252:2    0  79.5G  0 part 
```

### zc driver
Version:    

![/images/2023_11_30_10_00_30_392x183.jpg](/images/2023_11_30_10_00_30_392x183.jpg)

![/images/2023_11_30_12_13_37_701x371.jpg](/images/2023_11_30_12_13_37_701x371.jpg)

2019, error:    

![/images/2023_11_30_15_16_03_1048x405.jpg](/images/2023_11_30_15_16_03_1048x405.jpg)

uhd730 could be usable , but zC copy not ready.   

### win2022 way
Get the rbd(vdi node):    

```
# sudo virsh dumpxml privatedefaulttenant-default_bbdd760f-6709-4a0b-ad96-4903c6ea1e2e | grep rbd
      <source protocol='rbd' name='ceph-vm-pool-1/volume-b492ad15-e646-4cf3-9fc1-103d30756151'>

```
mount rbd(idv node):    

```
# rbd --id cinder map ceph-vm-pool-1/volume-b492ad15-e646-4cf3-9fc1-103d30756151 -p testpool2
/dev/rbd2
```
Start the machine
