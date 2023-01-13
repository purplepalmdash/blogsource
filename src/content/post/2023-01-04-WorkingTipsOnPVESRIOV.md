+++
title= "WorkingTipsOnPVESRIOV"
date = "2023-01-04T20:43:10+08:00"
description = "WorkingTipsOnPVESRIOV"
keywords = ["Technology"]
categories = ["Technology"]
+++
Change the repository for using chinese site:    

```
$ cat /etc/apt/sources.list
deb http://mirrors.ustc.edu.cn/debian bullseye main contrib

deb http://mirrors.ustc.edu.cn/debian bullseye-updates main contrib

# security updates
deb http://mirrors.ustc.edu.cn/debian-security bullseye-security main contrib
```
Update to 5.19 kernel and linux-header:    

```
deb http://mirrors.ustc.edu.cn/debian bullseye main contrib

deb http://mirrors.ustc.edu.cn/debian bullseye-updates main contrib

# security updates
deb http://mirrors.ustc.edu.cn/debian-security stable-security main contrib

# no-subscription
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription

```
Install kernel(via download):    

```
scp ./pve-headers-5.19.7-2-pve_5.19.7-2_amd64.deb pve-kernel-5.19.7-2-pve_5.19.7-2_amd64.deb root@192.168.1.52:~
root@xxxx:~# dpkg -i pve-headers-5.19.7-2-pve_5.19.7-2_amd64.deb pve-kernel-5.19.7-2-pve_5.19.7-2_amd64.deb 
reboot
```
Install `dkms`, then install `i915-sriov-dkms`.   

```
tar -xvf i915-sriov-dkms.tar
mv i915-sriov-dkms /usr/src
dkms install -m i915-sriov -v dkms
# dkms status
i915-sriov, dkms, 5.19.7-2-pve, x86_64: installed
```
Install sysfsutils:    

```
apt install sysfsutils -y
echo "devices/pci0000:00/0000:00:02.0/sriov_numvfs = 7" > /etc/sysfs.conf
```
