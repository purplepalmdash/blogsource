+++
title = "TryProxmox"
date = "2018-08-16T10:48:35+08:00"
description = "TryProxmox"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Installation
iso installation. 

![/images/2018_08_16_10_50_16_461x263.jpg](/images/2018_08_16_10_50_16_461x263.jpg)

### Configuration
Login:    

![/images/2018_08_16_10_54_19_398x203.jpg](/images/2018_08_16_10_54_19_398x203.jpg)
Image:    

![/images/2018_08_16_10_55_05_524x544.jpg](/images/2018_08_16_10_55_05_524x544.jpg)

Another node: 

![/images/2018_08_16_11_23_10_487x242.jpg](/images/2018_08_16_11_23_10_487x242.jpg)

### Cluster
Initial:    

![/images/2018_08_16_11_29_44_547x368.jpg](/images/2018_08_16_11_29_44_547x368.jpg)

Generation of the information:    

![/images/2018_08_16_11_33_06_799x254.jpg](/images/2018_08_16_11_33_06_799x254.jpg)
Join:    

![/images/2018_08_16_11_33_29_808x289.jpg](/images/2018_08_16_11_33_29_808x289.jpg)
Install system:    

![/images/2018_08_16_11_55_03_711x503.jpg](/images/2018_08_16_11_55_03_711x503.jpg)


### Command
After installation, build a cluster using CLI in following commands:    

Management node:    

```
# pvecm create mycluster
```
Working node, for joing:    

```
# pvecm add 192.168.0.121
```
Thus you will see the cluster being created as following:    

