+++
title = "Ovirt HyperConverged InAir-Gapped Environment"
date = "2020-02-14T14:42:39+08:00"
description = "OvirtHyperConvergedInAirGapedEnvironment"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 0. AIM
For deploying Ovirt HyperConverged in air-gapped environment.    
For some companies, their inner environment is air-gapped, e.g OA network. In
such air-gapped environment we could only use ISO and take some packages in
cd-roms for taking into their intra-network. How to deploy a ovirt drivened
private cloud in air-gapped room, I will take some experiment and try the
solution out.     
### 1. Environment
In this chapter the environment will be available for ovirt deployment with glusterfs.   
#### 1.1 Hardware
I use my home machine for building the environment, the hardware is listed as:    

```
CPU: Intel(R) Core(TM) i5-4460  CPU @ 3.20GHz
Memory: DDR3 1600 32G
Disk: 1T HDD.
```
#### 1.2 OS/Networking/Software
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

#### 1.3 VMs Preparation
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
engineinstance.com	10.20.30.34
```
For setting up the ip address, use `nmtui` in terminal, take instance1.com for example:    

![/images/2020_02_14_15_20_56_601x304.jpg](/images/2020_02_14_15_20_56_601x304.jpg)

For setting up the hostname, also use `nmtui`:    

![/images/2020_02_14_15_22_33_450x232.jpg](/images/2020_02_14_15_22_33_450x232.jpg)

Login to each machine and enable the password-less login, take instance1 for example:      

```
# ssh-keygen
# vim /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
10.20.30.31     instance1.com
10.20.30.32     instance2.com
10.20.30.33     instance3.com
10.20.30.34	engineinstance.com
# ssh-copy-id root@instance1.com
# ssh-copy-id root@instance2.com
# ssh-copy-id root@instance3.com
```
Also add following items(engine vm's hostname and ip address) into host machine(archLinux)'s `/etc/hosts`:      

```
10.20.30.31     instance1.com
10.20.30.32     instance2.com
10.20.30.33     instance3.com
10.20.30.34	engineinstance.com
```

### 2. Deploy Glusterfs
Use firefox for visiting `https://10.20.30.31:9090`:    

![/images/2020_02_14_15_39_07_796x287.jpg](/images/2020_02_14_15_39_07_796x287.jpg)

use root for login, enter the `instance1.com`'s cockpit web:    

![/images/2020_02_14_15_39_38_728x474.jpg](/images/2020_02_14_15_39_38_728x474.jpg)

Click `V`->`Hosted Engine`, then click the `start` button under `Hyperconverged`:    

![/images/2020_02_14_15_42_07_1040x611.jpg](/images/2020_02_14_15_42_07_1040x611.jpg)

Click `Run Gluster Wizard`:    

![/images/2020_02_14_15_43_46_665x134.jpg](/images/2020_02_14_15_43_46_665x134.jpg)

Fill in 3 nodes's hostname, click `next`:    

![/images/2020_02_14_15_45_01_890x403.jpg](/images/2020_02_14_15_45_01_890x403.jpg)

In `Additional Hosts`, click `Use same hostnames as in previous step`, thus Host2 and Hosts3 will be added automatically:    

![/images/2020_02_14_15_47_45_880x515.jpg](/images/2020_02_14_15_47_45_880x515.jpg)

In `Packages` we keep the default empty items and click next for continue.   

Keep the default volumn setting, and enable the Arbiter for `data` and `vmstore`:     

![/images/2020_02_14_15_57_02_843x459.jpg](/images/2020_02_14_15_57_02_843x459.jpg)

Here we adjust the LV device name to `vdb`, and adjust the size as `80,80,80`, click next for continue:     

The volume size for running engine vm should be at least 58GB(ovirt default minimum size, actually takes more than this number. )    

![/images/2020_02_14_16_00_34_809x611.jpg](/images/2020_02_14_16_00_34_809x611.jpg)

Review and click deploy:     

![/images/2020_02_14_16_02_53_870x648.jpg](/images/2020_02_14_16_02_53_870x648.jpg)

The ansible tasks will run until you see this hint:    

![/images/2020_02_14_16_09_18_602x375.jpg](/images/2020_02_14_16_09_18_602x375.jpg)

Click `Continue to hosted engine deployment` to continue.    

### 3. Hosted Engine
Before continue, manually install the rpms in `instance1.com`:       

```
# yum install -y ./ovirt-engine-appliance-4.3-20200127.1.el7.x86_64.rpm
# rpm -qa | grep ovirt-engine-appliance
ovirt-engine-appliance-4.3-20200127.1.el7.x86_64
```
Fill the engine vm's configuration infos:    

![/images/2020_02_14_16_23_51_518x881.jpg](/images/2020_02_14_16_23_51_518x881.jpg)

Fill in admin portal password(this password will be used in web login) and continue:    

![/images/2020_02_14_16_25_19_817x557.jpg](/images/2020_02_14_16_25_19_817x557.jpg)

Examine the configuration and click `Prepare VM`:    

![/images/2020_02_14_16_25_19_817x557.jpg](/images/2020_02_14_16_25_19_817x557.jpg)

Wait for about half an hour to see deployment successful:    

![/images/2020_02_14_17_00_27_713x388.jpg](/images/2020_02_14_17_00_27_713x388.jpg)   

Keep the default configuration:  

engine vm's storage configuration will use Gluster, path will be Gluster's
engine volumn, and its parameter is:    

`backup-volfile-servers=instance2.com:instance3.com`    

for preventing the single-node issue for Gluster.     

![/images/2020_02_14_17_17_52_772x455.jpg](/images/2020_02_14_17_17_52_772x455.jpg)

Click `Finish deployment`, and wait for a break:    

![/images/2020_02_14_17_20_30_817x433.jpg](/images/2020_02_14_17_20_30_817x433.jpg)

Seeing this means deploy succeeded:   

![/images/2020_02_14_17_40_22_591x535.jpg](/images/2020_02_14_17_40_22_591x535.jpg)

Refresh the status:     

![/images/2020_02_14_17_43_43_1058x513.jpg](/images/2020_02_14_17_43_43_1058x513.jpg)

### 4. Portal
Visit `engineinstance.com` in host machine(ArchLinux):     

![/images/2020_02_14_17_47_13_767x501.jpg](/images/2020_02_14_17_47_13_767x501.jpg)

Click `Administration Portal`:    

![/images/2020_02_14_17_48_30_499x308.jpg](/images/2020_02_14_17_48_30_499x308.jpg)

admin page is like following:     

![/images/2020_02_14_17_50_57_1221x561.jpg](/images/2020_02_14_17_50_57_1221x561.jpg)

ssh into engine vm and check the disk partitions:     

```
# ssh root@10.20.30.34
root@10.20.30.34's password:
Last login: Fri Feb 14 17:25:51 2020 from 192.168.1.1
[root@engineinstance ~]#df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 1.9G     0  1.9G   0% /dev
tmpfs                    1.9G   12K  1.9G   1% /dev/shm
tmpfs                    1.9G  8.9M  1.9G   1% /run
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/ovirt-root   8.0G  2.3G  5.8G  29% /
/dev/mapper/ovirt-home  1014M   33M  982M   4% /home
/dev/mapper/ovirt-tmp    2.0G   33M  2.0G   2% /tmp
/dev/mapper/ovirt-var     20G  437M   20G   3% /var
/dev/vda1               1014M  157M  858M  16% /boot
/dev/mapper/ovirt-log     10G   45M   10G   1% /var/log
/dev/mapper/ovirt-audit 1014M   34M  981M   4% /var/log/audit
tmpfs                    379M     0  379M   0% /run/user/0
```

### 5. Create The First VM
#### 5.1 Add ISO storage Domain
Login in to `instance1.com`, configure nfs share storage for holding ISO
images:    

```
[root@instance1 ]# mkdir -p /isoimages
[root@instance1 ]# chown 36:36 -R /isoimages/
[root@instance1 ]# chmod 0755 -R /isoimages/
[root@instance1 ]# vi /etc/exports
[root@instance1 ]# cat /etc/exports
/isoimages *(rw,sync,no_subtree_check,all_squash,anonuid=36,anongid=36)
[root@instance1 ]# systemctl enable --now  nfs.service   
Created symlink from /etc/systemd/system/multi-user.target.wants/nfs-server.service to /usr/lib/systemd/system/nfs-server.service.
```
In ovirt manager portal , click `Storage`->`Storage Domain`, click `New
Domain`:    

![/images/2020_02_14_19_29_49_1046x337.jpg](/images/2020_02_14_19_29_49_1046x337.jpg)

Fill in  name and path information:    

![/images/2020_02_14_19_42_32_1006x355.jpg](/images/2020_02_14_19_42_32_1006x355.jpg)

Finished adding isoimages:    

![/images/2020_02_14_19_32_52_919x263.jpg](/images/2020_02_14_19_32_52_919x263.jpg)

#### 5.2 Upload iso
Login to engien vm(engineinstance.com), download the iso from official site,
we take ubuntu16.04.6 for example:     

```
[root@engineinstance ~]# ovirt-iso-uploader -i isoimages upload ./ubuntu-16.04.6-server-amd64.iso 
Please provide the REST API password for the admin@internal oVirt Engine user (CTRL+D to abort): 
Uploading, please wait...
INFO: Start uploading ./ubuntu-16.04.6-server-amd64.iso 
Uploading: [########################################] 100%
INFO: ./ubuntu-16.04.6-server-amd64.iso uploaded successfully
```

#### 5.3 Create VM
Compute-> Virtual Machines, click `new` button:    

![/images/2020_02_14_19_48_14_846x313.jpg](/images/2020_02_14_19_48_14_846x313.jpg)
Fill in informations:    

![/images/2020_02_14_19_50_24_673x641.jpg](/images/2020_02_14_19_50_24_673x641.jpg)

Click advanced options, select `Boot Options`, then attach uploaded iso:    

![/images/2020_02_14_19_52_09_917x511.jpg](/images/2020_02_14_19_52_09_917x511.jpg)

Click Disks, then click `new`:    

![/images/2020_02_14_20_00_20_1178x316.jpg](/images/2020_02_14_20_00_20_1178x316.jpg)

Fill in options:    

![/images/2020_02_14_20_01_54_731x394.jpg](/images/2020_02_14_20_01_54_731x394.jpg)


Click this new machine, and select `run->run once`:    

![/images/2020_02_14_19_53_19_852x380.jpg](/images/2020_02_14_19_53_19_852x380.jpg)

Click OK for installation:    

![/images/2020_02_14_19_54_06_599x528.jpg](/images/2020_02_14_19_54_06_599x528.jpg)

The installation image will be shown:    

![/images/2020_02_14_19_55_49_641x534.jpg](/images/2020_02_14_19_55_49_641x534.jpg)

Configure installation options and wait until installation finished.     
Since we use nested virtualization, the installation step will take a very
long time(>1h) for installing the os. For speedup, considering use NVME ssd
for locating the vm's qcow2 files. Or use 3 physical servers.     

On vm portal we could see our newly created vm:    

![/images/2020_02_14_21_03_33_764x485.jpg](/images/2020_02_14_21_03_33_764x485.jpg)

Examine the vms on instance1.com:     

```
[root@instance1 isoimages]# virsh -r list
 Id    Name                           State
----------------------------------------------------
 2     HostedEngine                   running
 4     ubuntu1604                     running
```

### 6. Create vm using template
#### 6.1 Create template
Create template via:    

![/images/2020_02_14_21_49_05_743x666.jpg](/images/2020_02_14_21_49_05_743x666.jpg)

Check the status of template:    

![/images/2020_02_14_21_50_02_733x224.jpg](/images/2020_02_14_21_50_02_733x224.jpg)

#### 6.2 Create vm
Create new vm using template:    

![/images/2020_02_14_21_57_36_912x668.jpg](/images/2020_02_14_21_57_36_912x668.jpg)

Start the machine and check result:    

![/images/2020_02_14_22_05_30_1148x360.jpg](/images/2020_02_14_22_05_30_1148x360.jpg)

### 7. Add hosts
In engine vm, add following items:    

```

```

Then we add hosts of `instance2.com` and `instance3.com`:    

![/images/2020_02_14_22_16_23_904x687.jpg](/images/2020_02_14_22_16_23_904x687.jpg)

Result:    

![/images/2020_02_14_22_17_48_860x208.jpg](/images/2020_02_14_22_17_48_860x208.jpg)

