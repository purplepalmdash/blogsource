+++
title= "ltspbaremetalInstall"
date = "2024-12-05T09:59:05+08:00"
description = "ltspbaremetalInstall"
keywords = ["Technology"]
categories = ["Technology"]
+++
Hardware/OS/Software info:    

```
dash@i9server:~$ cat /etc/issue
Ubuntu 22.04.5 LTS \n \l

dash@i9server:~$ uname -a
Linux i9server 6.8.0-49-generic #49~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Wed Nov  6 17:42:15 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
dash@i9server:~$ lsblk
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0         7:0    0    64M  1 loop /snap/core20/2379
loop1         7:1    0  63.7M  1 loop /snap/core20/2434
loop2         7:2    0    87M  1 loop /snap/lxd/29351
loop3         7:3    0  89.4M  1 loop /snap/lxd/31333
loop4         7:4    0  38.8M  1 loop /snap/snapd/21759
loop5         7:5    0  44.3M  1 loop /snap/snapd/23258
sda           8:0    0   1.8T  0 disk 
└─sda1        8:1    0   1.8T  0 part 
sr0          11:0    1  1024M  0 rom  
nvme0n1     259:0    0 476.9G  0 disk 
├─nvme0n1p1 259:1    0     1G  0 part /boot/efi
└─nvme0n1p2 259:2    0 475.9G  0 part /
dash@i9server:~$ free -m
               total        used        free      shared  buff/cache   available
Mem:           64079         565       60533           2        2981       62859
Swap:           8191           0        8191
dash@i9server:~$ cat /proc/cpuinfo | grep -i model
model		: 165
model name	: Intel(R) Core(TM) i9-10900 CPU @ 2.80GHz
```
Install ltsp:    

```
sudo add-apt-repository ppa:ltsp
sudo apt update
sudo apt install --install-recommends ltsp ltsp-binaries dnsmasq nfs-kernel-server openssh-server squashfs-tools ethtool net-tools epoptes
sudo gpasswd -a dash epoptes
```
Prepare the image:    

```
# ls *.img -l -h
-rw-r--r-- 1 dash dash 4.0G Dec  5 02:33 kylin.img
-rw-r--r-- 1 dash dash 2.6G Dec  5 02:35 ubuntu2004.img
-rw-r--r-- 1 dash dash 3.2G Dec  5 02:34 uos613.img
-rw-r--r-- 1 dash dash 4.4G Dec  5 02:34 zkfd613.img
```
Create the dns/dhcp, import the images:    

```
ltsp dnsmasq --proxy-dhcp=0
vim  /etc/dnsmasq.d/ltsp-dnsmasq.conf
    dhcp-range=192.168.1.34,192.168.1.250,600h
mkdir /srv/ltsp && cd /srv/ltsp
ln -s /home/dash/ubuntu2004.img .
ltsp image ubuntu2004
ltsp ipxe
ltsp nfs
    Installed /usr/share/ltsp/server/nfs/ltsp-nfs.exports in /etc/exports.d/ltsp-nfs.exports
ltsp initrd
useradd -m test1
useradd -m test2
passwd test1
passwd test2

```  
more images:    

```
 rm -f *.img
 ln -s /home/dash/uos613.img .
 ltsp image uos613 && ltsp ipxe && ltsp initrd
 rm -f *.img
 ln -s /home/dash/zkfd613.img .
 ltsp image zkfd613 && ltsp ipxe && ltsp initrd
 rm -f *.img
 ln -s /home/dash/kylin.img .
 ltsp image kylin && ltsp ipxe && ltsp initrd
```
Result: zkfd behaves bad  


deepin image:    

```
cd /srv/ltsp
ln -s /var/lib/libvirt/trueimages/deepin.img .
ltsp image deepin && ltsp ipxe && ltsp initrd

``` 
