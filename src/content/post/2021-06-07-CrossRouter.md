+++
title= "CrossRouter"
date = "2021-06-07T07:36:04+08:00"
description = "CrossRouter"
keywords = ["Technology"]
categories = ["Technology"]
+++
In openwrt, configure following:    

![/images/2021_06_07_07_36_24_553x374.jpg](/images/2021_06_07_07_36_24_553x374.jpg)

Then in nuc(`192.168.1.222`) add following items:   

```
# sudo route add -net 192.168.0.17 netmask 255.255.255.255 gw 192.168.1.2 enp3s0
# sudo route add -net 192.168.0.0 netmask 255.255.255.0 gw 192.168.1.2 enp3s0
```
While `192.168.1.2` is the openWRT's wan address, while the 192.168.0.0/24 is its lan address range.   

From now we could directly get to the `192.168.0.1/24` range.   
