+++
categories = ["iscsi"]
date = "2016-08-23T18:06:13+08:00"
description = "On Setup FreeNAS For CloudStack"
keywords = ["iscsi"]
title = "OnFreeNASAndCloudStack"

+++
### iscsi service
Enable it via:     

![/images/2016_08_23_18_06_58_278x449.jpg](/images/2016_08_23_18_06_58_278x449.jpg)    

Configuration steps could be refers to:    

[http://purplepalmdash.github.io/2015/07/17/iscsi-installed-debian-jessie/](http://purplepalmdash.github.io/2015/07/17/iscsi-installed-debian-jessie/)   

### Integration
Discover the iscsi storage via:    

```
# iscsiadm -m discovery -t st -p 192.168.10.5:3260
192.168.10.5:3260,-1 iqn.xxxxxx.iscsi:cloudstackiscsi
``` 
Then you could add it into cloudstack via:    

![/images/2016_08_23_20_25_24_433x634.jpg](/images/2016_08_23_20_25_24_433x634.jpg)   

Add a service offering which tag is `iscsi`, then start the instance with this tagged
service offering.   

### Test
Find and login with following command:    

```
# iscsiadm -m discovery -t st -p 192.168.10.5:3260
# iscsiadm -m node -T iqn.xxxxx.iscsi:forlinux -p 192.168.10.5:3260 -l
```  
Use fio for testing:    

```
# fio --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=/dev/sdb
--bs=4k --iodepth=2 --size=125G --readwrite=randread --runtime=1800
```
