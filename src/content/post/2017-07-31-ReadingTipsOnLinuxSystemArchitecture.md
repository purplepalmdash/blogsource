+++
title = "ReadingTipsOnLinuxSystemArchitecture"
date = "2017-07-31T09:19:01+08:00"
description = "ReadingTipsOnLinuxSystemArchitecture"
keywords = ["Linux"]
categories = ["Linux"]
+++
### On This Book
Borrowed from lab, written via a janpanese author.    
![/images/2017_07_31_09_20_33_1054x739.jpg](/images/2017_07_31_09_20_33_1054x739.jpg)
This article will record the reading tips on Chapter 2(libvirtd related).    
### Network Configuration 
Edit the netoworking definition xml:    

```
$ cat internal.xml
<network>
	<name>internal</name>
	<bridge name='virbr8'/>
</network>
$  cat external.xml
<network>
	<name>external</name>
	<bridge name='virbr9'/>
</network>
```
Define the networking via following commands:    

```
$ sudo virsh net-define external.xml
Network external defined from external.xml

$ sudo virsh net-autostart external
Network external marked as autostarted

$ sudo virsh net-start external
Network external started

$ libvirt sudo virsh net-list
 Name                 State      Autostart     Persistent
----------------------------------------------------------
 default              active     no            yes
 external             active     yes           yes
 internal             active     yes           yes
 kubernetes           active     yes           yes
```
View the configuration in virt-manager:    

![/images/2017_07_31_09_34_07_495x298.jpg](/images/2017_07_31_09_34_07_495x298.jpg)

