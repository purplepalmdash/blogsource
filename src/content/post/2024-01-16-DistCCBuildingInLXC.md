+++
title= "DistCCBuildingInLXC"
date = "2024-01-16T15:57:43+08:00"
description = "DistCCBuildingInLXC"
keywords = ["Technology"]
categories = ["Technology"]
+++
Create a profile named `bridgeprofile`:     

```
$ lxc profile create bridgeprofile
$  lxc profile show bridgeprofile
config: {}
description: Bridged networking LXD profile
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: br0
    type: nic
name: bridgeprofile
```
Steps:     

```
$ cat bridge
config: {}
description: Bridged networking LXD profile
devices:
  eth0:
    name: eth0
    nictype: bridged
    parent: br0
    type: nic
name: bridgeprofile
$ cat bridge | lxc profile edit bridgeprofile
```
Then create a lxc named distcc:     

```
$ lxc launch -p default -p bridgeprofile ubuntu:22.04 distcc
```
Edit the netplan confguration in lxc instance(`distcc`):      


```
root@distcc:~# cat /etc/netplan/50-cloud-init.yaml 
network:
    version: 2
    ethernets:
        eth0:
            dhcp4: false
            addresses: [192.168.1.9/24]
            gateway4: 192.168.1.33
root@distcc:~# netplan
```
Install distcc via:    

```
apt install -y distcc build-essential
```
Configure the distcc:    

```
STARTDISTCC="true"
ALLOWEDNETS="192.168.1.0/24"
LISTENER="0.0.0.0"
JOBS="12"
```

