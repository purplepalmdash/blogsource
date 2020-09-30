+++
title= "OpenWRTBox"
date = "2020-09-28T09:43:16+08:00"
description = "OpenWRTBox"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Tips
Version: ` ATTITUDE ADJUSTMENT (12.09, r36088)`, so we have to login into this box via:    

```
$ ssh -oKexAlgorithms=+diffie-hellman-group1-sha1  root@192.168.2.1
```
Luckly we got the dhcp enabled in this equipment!!! Otherwise this equipment is bricked.  

Change wifi setting under `Network->Wifi->Interface Configuration->General Setup`:    

![/images/2020_09_28_09_49_31_986x277.jpg](/images/2020_09_28_09_49_31_986x277.jpg)
 
