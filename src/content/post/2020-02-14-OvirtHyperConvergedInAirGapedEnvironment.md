+++
title = "Ovirt HyperConverged InAir-Gapped Environment"
date = "2020-02-14T14:42:39+08:00"
description = "OvirtHyperConvergedInAirGapedEnvironment"
keywords = ["Linux"]
categories = ["Linux"]
+++
### AIM
For deploying Ovirt HyperConverged in air-gapped environment.    
For some companies, their inner environment is air-gapped, e.g OA network. In
such air-gapped environment we could only use ISO and take some packages in
cd-roms for taking into their intra-network. How to deploy a ovirt drivened
private cloud in air-gapped room, I will take some experiment and try the
solution out.     
### Hardware
I use my home machine for building the environment, the hardware is listed as:    

```
CPU: Intel(R) Core(TM) i5-4460  CPU @ 3.20GHz
Memory: DDR3 1600 32G
Disk: 1T HDD.
```
### OS/Networking/Software
My home machine runs ArchLinux, with nested virtualization.    
Use qemu and virt-manager for setting the environment.    

```
# qemu-system-x86_64 --version                                                                                                           
QEMU emulator version 4.2.0
Copyright (c) 2003-2019 Fabrice Bellard and the QEMU Project developers
# virt-manager --version
2.2.1
```
I setup a isolated networking in virt-manager, cidr is 10.20.30.0/24, 3 vms
will use this isolated networking for emulating the air-gapped environment, its name is `ovirt-isolated`:    

![/images/2020_02_14_14_51_10_545x560.jpg](/images/2020_02_14_14_51_10_545x560.jpg)

### VMS
I use 3 vms for setting up the environment, each of them have:    

```
2 vcpus
10240 MB memory
vda: 100 GB, for installing the system. 
vdb: 300 GB, for setting up the storage network.   
NIC: 1x, attached to ovirt-isolated networking. 
```
hostname - IP is listed as following:     

```
instance1.com	10.20.30.31
instance2.com	10.20.30.32
instance3.com	10.20.30.33
```
