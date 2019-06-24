+++
title = "VmwareBasedKubeflow"
date = "2019-06-20T14:34:06+08:00"
description = "VmwareBasedKubeflow"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Vmware
Download and install Vmware workstation pro on Ubuntu:    

```
# wget https://download3.vmware.com/software/wkst/file/VMware-Workstation-Full-15.1.0-13591040.x86_64.bundle
# chmod 777 *.bundle
# ./VMware-Workstation-Full-15.1.0-13591040.x86_64.bundle
```
### VM Installation
Create a new machine:    

![/images/2019_06_20_14_48_31_623x305.jpg](/images/2019_06_20_14_48_31_623x305.jpg)

Select iso:    

![/images/2019_06_20_14_49_11_688x284.jpg](/images/2019_06_20_14_49_11_688x284.jpg)

Personalize Linux:    

![/images/2019_06_20_14_50_42_394x207.jpg](/images/2019_06_20_14_50_42_394x207.jpg)

Select location:    

![/images/2019_06_20_14_51_23_531x189.jpg](/images/2019_06_20_14_51_23_531x189.jpg)

Specify processor:    

![/images/2019_06_20_14_51_44_313x135.jpg](/images/2019_06_20_14_51_44_313x135.jpg)

Specify Memory:    

![/images/2019_06_20_14_52_14_506x278.jpg](/images/2019_06_20_14_52_14_506x278.jpg)

Specify NAT:   

![/images/2019_06_20_14_52_39_522x207.jpg](/images/2019_06_20_14_52_39_522x207.jpg)

I/O Controller:    

![/images/2019_06_20_14_52_58_399x153.jpg](/images/2019_06_20_14_52_58_399x153.jpg)

Virtual Disk Type:    

![/images/2019_06_20_14_53_15_255x146.jpg](/images/2019_06_20_14_53_15_255x146.jpg)

Create a new disk:    

![/images/2019_06_20_14_53_31_534x229.jpg](/images/2019_06_20_14_53_31_534x229.jpg)

Specify 200GB:   

![/images/2019_06_20_14_53_48_494x288.jpg](/images/2019_06_20_14_53_48_494x288.jpg)

Disk File:    

![/images/2019_06_20_14_54_01_508x152.jpg](/images/2019_06_20_14_54_01_508x152.jpg)

Click Finish and begin installation.    

When installation, click poweroff, and setting , remove the auto install iso:    

![/images/2019_06_20_14_56_21_575x368.jpg](/images/2019_06_20_14_56_21_575x368.jpg)

Uncheck `Connect at power on` for both Floppy and CD/DVD(SATA), then reboot, you will see 
our customized installation items, use them for installing:    

![/images/2019_06_20_14_56_49_623x327.jpg](/images/2019_06_20_14_56_49_623x327.jpg)

After installation, check the configured ip address:    

![/images/2019_06_20_15_10_47_579x306.jpg](/images/2019_06_20_15_10_47_579x306.jpg)

### System
Default grub configuration:

```
# vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="net.ifnames=0 biosdevname=0"
# grub-mkconfig -o /boot/grub/grub.cfg
# vim /etc/netplan/01-netcfg.yaml
    eth0:
      dhcp4: no
      addresses: [172.16.107.17/24]
      gateway4: 172.16.107.2
```
Then reboot and use our ansible code for deploying.   

### Make it Online
Remove the dns items for `docker.io, quay.io, gcr.io, elastic.co`:    

`vim /etc/bind/named.conf.default-zones`:    

![/images/2019_06_20_16_49_55_433x490.jpg](/images/2019_06_20_16_49_55_433x490.jpg)

logout from our fake website:    

```
root@node0:/home/test# docker logout quay.io
Removing login credentials for quay.io
root@node0:/home/test# docker logout gcr.io
Removing login credentials for gcr.io
root@node0:/home/test# docker logout k8s.gcr.io
Removing login credentials for k8s.gcr.io
root@node0:/home/test# docker logout docker.elastic.co
Removing login credentials for docker.elastic.co
root@node0:/home/test# docker logout
```
Remove the crt files and update the ca certification:    

```
# cd /usr/local/share/ca-certificates/
# mv server.crt /root
# update-ca-certificates 
# cd /etc/ssl/
# cd certs/
# rm -f server.pem 
# update-ca-certificates 
```
Then remove the items for bind9.     

Now reboot the machine.   
