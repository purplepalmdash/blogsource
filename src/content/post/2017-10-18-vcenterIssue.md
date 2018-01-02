+++
title = "vcenterIssue"
date = "2017-10-18T09:11:27+08:00"
description = "vcenterIssue"
keywords = ["Linux"]
categories = ["Linux"]
+++
### AIM
enlarge the disk space
### vCenter Password
For changing the vCenter password, do following:    
First you should get the vCenter vm running on which machine:    

![/images/2017_10_18_09_12_48_341x359.jpg](/images/2017_10_18_09_12_48_341x359.jpg)

Now use vsphere client login to `192.192.192.70`, open the terminal window,
then edit the grub items for setting the parameter to `init=/bin/bash`, now
you got a window which you could change the passwd of root.    

### Enlarge disk
Enlarge it on vsphere client, then run following command:    

```
# vpxd_servicecfg storage lvm autogrow
```
