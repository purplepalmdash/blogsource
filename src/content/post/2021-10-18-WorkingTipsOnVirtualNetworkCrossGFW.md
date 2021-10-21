+++
title= "WorkingTipsOnVirtualNetworkCrossGFW"
date = "2021-10-18T10:36:34+08:00"
description = "WorkingTipsOnVirtualNetworkCrossGFW"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Environment Description
In virt-manager, create a new isolated virtual network via:    

![/images/2021_10_18_10_39_01_522x540.jpg](/images/2021_10_18_10_39_01_522x540.jpg)


### Create VM
Get the openwrt x86 images from:    

```
wget https://downloads.openwrt.org/releases/21.02.0-rc4/targets/x86/64/openwrt-21.02.0-rc4-x86-64-generic-ext4-combined.img.gz
gunzip openwrt-21.02.0-rc4-x86-64-generic-ext4-combined.img.gz 
mv /root/openwrt-21.02.0-rc4-x86-64-generic-ext4-combined.img /var/lib/libvirt/images
```
Create a new virtual machine using this image:    

![/images/2021_10_18_11_32_10_487x492.jpg](/images/2021_10_18_11_32_10_487x492.jpg)

2-Core , 1024 MB:    

![/images/2021_10_18_11_32_30_291x173.jpg](/images/2021_10_18_11_32_30_291x173.jpg)

Named it `zzzz_OpenWRT`:    

![./images/2021_10_18_11_32_44_354x265.jpg](./images/2021_10_18_11_32_44_354x265.jpg)

Customize the vm, first define an ethernet card which use `default` network:    

![/images/2021_10_18_11_33_46_449x255.jpg](/images/2021_10_18_11_33_46_449x255.jpg)

Add a second nic which uses `isolated` network:    

![/images/2021_10_18_11_34_20_611x214.jpg](/images/2021_10_18_11_34_20_611x214.jpg)

Bootup the machine until you see:     

![/images/2021_10_18_11_35_07_584x285.jpg](/images/2021_10_18_11_35_07_584x285.jpg)

You should use `passwd` for generating a new password for security.  

### Configure
Swith the eth0/eth1, cause eth0 will be the lan, while eth1 will be the wan.   

![/images/2021_10_18_11_49_52_617x258.jpg](/images/2021_10_18_11_49_52_617x258.jpg)

Manually set the br-lan address via:    

```
# vim /etc/config/network
- option ipaddr '192.168.1.1'
+ option ipaddr '10.17.18.254'
```
Login luci via:    

![/images/2021_10_18_11_52_08_714x545.jpg](/images/2021_10_18_11_52_08_714x545.jpg)

Install redsocks via:    

```
# opkg update
# opkg install redsocks
```
