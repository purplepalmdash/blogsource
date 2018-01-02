+++
categories = ["Virtualization"]
date = "2016-08-10T14:29:27+08:00"
description = "Setup CloudStack Site to Site VPN"
keywords = ["CloudStack"]
title = "CloudStackSiteToSiteVPNHowTo"

+++
Recently I am investigating the VPN solution on company's private cloud. Some guys
insist on setting up the site-to-site VPN on Juniper's firewall(SR240), but I think
this is a piece of shit. Setting up the VPN on physical firewall will lose the `ease of
use` and `ease of configuration` of CloudStack's built-in VPN functionality, so
following are the steps for me to investigate the CloudStack's site-to-site VPN among
different vpcs. Follow the guide line you will get two or more geographical independent
sites working as in a big ethernet.    

### Prerequisites
The environment is pretty simple, you need 2 PCs(Or more) for experiment.    

![/images/2016_08_10_14_45_50_603x334.jpg](/images/2016_08_10_14_45_50_603x334.jpg)    

Working Machine 1's configuration:    

```
Intel(R) Core(TM) i7-3770 CPU @ 3.40GHz
24 G Memory
1000M Ethernet Card
More than 20G Free disk    
ArchLinux
```

Working Machine 2's configuration:    

```
Intel(R) Core(TM) i3 CPU         540  @ 3.07GHz
8 G Memory
1000M Ethernet Card
More than 20G Free disk    
ArchLinux
```
Make sure the nested KVM is configurated in both machine:    

```
$ systool -m kvm_intel -v | grep nested
    nested              = "Y"
```
Also make sure you have a Linux Bridge(Not the physical card or OpenVswitch Bridge!).   

### Testing VM
We create two virtual machines for installing cloudstack all in one environment, like
following picture.   

![/images/2016_08_10_14_53_23_652x427.jpg](/images/2016_08_10_14_53_23_652x427.jpg)     

Virtual Machine 1:    

```
4 core(host-passthrough CPU)
8G Memory
200G Disk
Bridged to br0
CentOS 6.8 X86_64
```

Virtual Machine 2(Because working Machine 2 only have 8G memory, we allocated 7G to
it):        

```
4 core(host-passthrough CPU)
7G Memory
200G Disk
Bridged to br0
CentOS 6.8 X86_64
```

The network configuration is listed as:    

![/images/2016_08_10_14_56_33_689x188.jpg](/images/2016_08_10_14_56_33_689x188.jpg)    

Specified `shared` is very important, or you won't get networking works.    

We named VirtualMachine1 to `vpn1452`, VirtualMachine2 to `vpn2452`, because we do the
investigation based on CloudStack 4.5.2 version.    

### vpn1452
vpn1452's IP Address is `172.16.0.2`, we started from the CloudStack management
webpage:    

![/images/2016_08_10_15_45_14_487x432.jpg](/images/2016_08_10_15_45_14_487x432.jpg)    
Click `I have used cloudstack before`, continue to configurate the environment.    

Change the options of following:    

Secondary storage allow(value: 172.16.0.0/16):    

![/images/2016_08_10_15_47_49_1140x252.jpg](/images/2016_08_10_15_47_49_1140x252.jpg)   

Local Storage, systemvm.use to true:    

![/images/2016_08_10_15_49_05_1022x371.jpg](/images/2016_08_10_15_49_05_1022x371.jpg)    

Save the configuration via restart the cloudstack-management service `service
cloudstack-management restart`     

Create a new zone(Select Advanced zone):   

![/images/2016_08_10_15_51_37_642x407.jpg](/images/2016_08_10_15_51_37_642x407.jpg)    

Zone Name and DNS:    

![/images/2016_08_10_15_52_40_417x464.jpg](/images/2016_08_10_15_52_40_417x464.jpg)    

Select hypervisor and enable local storages:    

![/images/2016_08_10_15_53_32_472x345.jpg](/images/2016_08_10_15_53_32_472x345.jpg)    

Public Networking configuration:  

![/images/2016_08_10_15_55_01_614x108.jpg](/images/2016_08_10_15_55_01_614x108.jpg)     

Hit add and hit next:    

![/images/2016_08_10_15_55_38_637x132.jpg](/images/2016_08_10_15_55_38_637x132.jpg)     

A New pod named `pod1452`:    

![/images/2016_08_10_15_56_46_472x404.jpg](/images/2016_08_10_15_56_46_472x404.jpg)   

Guest Traffic :    

![/images/2016_08_10_15_57_24_367x202.jpg](/images/2016_08_10_15_57_24_367x202.jpg)

Cluster Name:    

![/images/2016_08_10_15_58_05_495x100.jpg](/images/2016_08_10_15_58_05_495x100.jpg)   

Add Host(172.16.0.2):    

![/images/2016_08_10_15_58_57_428x294.jpg](/images/2016_08_10_15_58_57_428x294.jpg)   

Secondary storage:    

![/images/2016_08_10_15_59_44_433x211.jpg](/images/2016_08_10_15_59_44_433x211.jpg)      

Hit next and finally enable the zone:     

![/images/2016_08_10_16_01_22_508x449.jpg](/images/2016_08_10_16_01_22_508x449.jpg)   

Wait for a while to see the infrastucture works properly.   

#### Create VPC
Hit Network-> VPC, `Add VPC`:    

![/images/2016_08_10_16_06_21_760x141.jpg](/images/2016_08_10_16_06_21_760x141.jpg)   

 Add a new VPC like following:    

![/images/2016_08_10_16_07_07_405x442.jpg](/images/2016_08_10_16_07_07_405x442.jpg)   

Wait for VR start up, when VR becomes ready, add a new tier via clicking `Configure`
button:    

![/images/2016_08_10_16_08_39_662x145.jpg](/images/2016_08_10_16_08_39_662x145.jpg)   

Configuration:    

![/images/2016_08_10_16_09_35_377x324.jpg](/images/2016_08_10_16_09_35_377x324.jpg)   

Your new vpc and tier will be viewed as following:    

![/images/2016_08_10_16_10_14_712x412.jpg](/images/2016_08_10_16_10_14_712x412.jpg)   

Add an instance which in the new tier,   

![/images/2016_08_10_16_11_37_638x254.jpg](/images/2016_08_10_16_11_37_638x254.jpg)   

Examine the instance's ip configuration:    

![/images/2016_08_10_16_13_21_340x313.jpg](/images/2016_08_10_16_13_21_340x313.jpg)   

### vpn2452
The steps are mainly the same as in vpn1452, but with the following parameters:    

```
zone name: zone2452
public: 10.168.72.110~10.168.72.129
podname: pod2452
pod ip: 172.16.0.130~172.16.0.149
host: 172.16.0.3
``` 

#### vpc2452
Configuration for vpc2452:    

![/images/2016_08_10_16_29_05_383x430.jpg](/images/2016_08_10_16_29_05_383x430.jpg)   

tier 2452:    

![/images/2016_08_10_16_30_21_398x340.jpg](/images/2016_08_10_16_30_21_398x340.jpg)   

Now use a instance for testing the tier2452:    

![/images/2016_08_10_16_34_15_456x306.jpg](/images/2016_08_10_16_34_15_456x306.jpg)   

Without site-to-site vpn, testing ping:    

![/images/2016_08_10_16_35_24_380x180.jpg](/images/2016_08_10_16_35_24_380x180.jpg)   

### Site-to-Site VPN
We will configure both side, first we create vpc1452:    
#### vpc1452
Fist use following command for generating the key:    

```
# dd if=/dev/random count=16 bs=1 | xxd -ps
```
Record the key, for later on vpc2452 we will use the same key for configurating the
site-to-site VPN.    

Click vpc's configure button, to get the configuration window:    

![/images/2016_08_10_16_43_48_600x145.jpg](/images/2016_08_10_16_43_48_600x145.jpg)    

Click the `site-to-site VPNS`, a VPN Gateway will be generated:    

![/images/2016_08_10_16_44_51_568x151.jpg](/images/2016_08_10_16_44_51_568x151.jpg)   

Create a VPN Customer Gateway, like following configuration:    

![/images/2016_08_10_16_46_14_352x511.jpg](/images/2016_08_10_16_46_14_352x511.jpg)   

Notice, the Gateway`10.168.72.112` is generated in the same procedure in vpn2452 side.
the CIDR list is the vpn2452 side's tier ip ranges, ipsec preshared-key is gerated via
`dd` commands.     

Now a new `vpnto2452` vpn customer gateway is generated, 

![/images/2016_08_10_16_48_17_661x142.jpg](/images/2016_08_10_16_48_17_661x142.jpg)       

Back to vpc window, site-to-site vpn, create a new vpn connection:    

![/images/2016_08_10_16_49_18_683x123.jpg](/images/2016_08_10_16_49_18_683x123.jpg)    

The configuration for the VPN connection is listed as: notice we must select Passive:    

![/images/2016_08_10_16_50_11_341x185.jpg](/images/2016_08_10_16_50_11_341x185.jpg)    

Now the created VPN Connection is like following, remains state `Disconnected` because
it will be triggered via VPN2452's dial-in.    

![/images/2016_08_10_16_51_16_692x152.jpg](/images/2016_08_10_16_51_16_692x152.jpg)   

#### vpc2452
The vpc and `site-to-site VPNS` configuration is the same as vpc1452, for VPN Customer
Gateway, the configuration is listed :    

![/images/2016_08_10_16_57_50_364x533.jpg](/images/2016_08_10_16_57_50_364x533.jpg)   

a new VPN Customer Gateway is created as:    

![/images/2016_08_10_16_58_45_702x166.jpg](/images/2016_08_10_16_58_45_702x166.jpg)   

Then back to `site-to-site` VPN connection, create a new VPN connection:    

![/images/2016_08_10_16_59_41_393x216.jpg](/images/2016_08_10_16_59_41_393x216.jpg)    

DO NOT select passive, because this one will be activate the remote(vpc1452)'s VPN
connection.    

A new connection will be setup:    

![/images/2016_08_10_17_00_48_734x210.jpg](/images/2016_08_10_17_00_48_734x210.jpg)   

Now the created instance in vpc2452 will get reach with the instance which runs in vpc1452:    

![/images/2016_08_10_17_02_09_356x288.jpg](/images/2016_08_10_17_02_09_356x288.jpg)      

![/images/2016_08_10_17_03_02_347x138.jpg](/images/2016_08_10_17_03_02_347x138.jpg)    

You can login to the VR for examing the ipsec processes:    

![/images/2016_08_10_17_04_17_604x317.jpg](/images/2016_08_10_17_04_17_604x317.jpg)    

#### Back to vpc1452
Now the site-to-site vpn status has changed to:    

![/images/2016_08_10_17_05_27_766x177.jpg](/images/2016_08_10_17_05_27_766x177.jpg)   

### Conclusion
The cloudstack's site-to-site VPN functionality is very easy to use. Use it we could
easily setup the connections between different VPCs across Internet.    

Later I will show how to setup the site-to-site VPN with OpenStack and other platforms.       
