+++
title= "WorkingTipsOnLXDMigration"
date = "2021-07-21T09:54:23+08:00"
description = "WorkingTipsOnLXDMigration"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 环境
两台虚拟机用来模拟现实环境中的双节点（可以拓展到多节点）场景下的物理机器关机导致的LXD容器的迁移情况。   

机器配置（以mig2为例):     

```
root@mig2:/home/test# cat /etc/issue
Ubuntu 20.04.2 LTS \n \l
root@mig2:/home/test# free -g
              total        used        free      shared  buff/cache   available
Mem:              9           0           8           0           0           9
Swap:             0           0           0
root@mig2:/home/test# lxd --version
4.0.7
```

### 步骤
mig1节点上初始化:    

```
root@mig1:/home/test# lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: yes
What IP address or DNS name should be used to reach this node? [default=192.168.89.11]: 
Are you joining an existing cluster? (yes/no) [default=no]: no
What name should be used to identify this node in the cluster? [default=mig1]: 
Setup password authentication on the cluster? (yes/no) [default=no]: yes
Trust password for new clients: 
Again: 
Do you want to configure a new local storage pool? (yes/no) [default=yes]: 
Name of the storage backend to use (lvm, zfs, btrfs, dir) [default=zfs]: 
Create a new ZFS pool? (yes/no) [default=yes]: 
Would you like to use an existing empty block device (e.g. a disk or partition)? (yes/no) [default=no]: 
Size in GB of the new loop device (1GB minimum) [default=30GB]: 
Do you want to configure a new remote storage pool? (yes/no) [default=no]: 
Would you like to connect to a MAAS server? (yes/no) [default=no]: 
Would you like to configure LXD to use an existing bridge or host interface? (yes/no) [default=no]: 
Would you like to create a new Fan overlay network? (yes/no) [default=yes]: 
What subnet should be used as the Fan underlay? [default=auto]: 
Would you like stale cached images to be updated automatically? (yes/no) [default=yes] 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]:
```
mig2节点加入集群:   

```
root@mig2:/home/test# lxd init
Would you like to use LXD clustering? (yes/no) [default=no]: yes
What IP address or DNS name should be used to reach this node? [default=192.168.89.12]: 
Are you joining an existing cluster? (yes/no) [default=no]: yes
Do you have a join token? (yes/no) [default=no]: no
What name should be used to identify this node in the cluster? [default=mig2]: 
IP address or FQDN of an existing cluster node: 192.168.89.11
Cluster fingerprint: 75ee6a1962985e0262d6bea9f95d554f197719cca19820671856280fe0d2e28b
You can validate this fingerprint by running "lxc info" locally on an existing node.
Is this the correct fingerprint? (yes/no) [default=no]: yes
Cluster trust password: 
All existing data is lost when joining a cluster, continue? (yes/no) [default=no] yes
Choose "zfs.pool_name" property for storage pool "local": 
Choose "size" property for storage pool "local": 30GB
Choose "source" property for storage pool "local": 
Would you like a YAML "lxd init" preseed to be printed? (yes/no) [default=no]: 
root@mig2:/home/test# 
```
创建成功后，检查cluster情况:    

```
# root@mig2:/home/test# lxc cluster list
To start your first instance, try: lxc launch ubuntu:18.04

+------+----------------------------+----------+--------+-------------------+--------------+
| NAME |            URL             | DATABASE | STATE  |      MESSAGE      | ARCHITECTURE |
+------+----------------------------+----------+--------+-------------------+--------------+
| mig1 | https://192.168.89.11:8443 | YES      | ONLINE | Fully operational | x86_64       |
+------+----------------------------+----------+--------+-------------------+--------------+
| mig2 | https://192.168.89.12:8443 | YES      | ONLINE | Fully operational | x86_64       |
+------+----------------------------+----------+--------+-------------------+--------------+

```
