+++
title = "EnableIPMIOnNetdata"
date = "2020-01-03T10:46:38+08:00"
description = "EnableIPMIOnNetdata"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Prerequites
Build netdata from source, generate deb/rpm.    
Install following packages:    

```
lm-sensors libsensors4 libsensors4-dev freeipmi-tools freeipmi libfreeipmi16
```
### Configuration
Enable the freeipmi plugins:    

```
# vim /etc/netdata/netdata.conf
[plugins]
#cgroups = no
freeipmi = yes

[plugin:freeipmi]
    update every = 5
    command options = 
```

Change the permission of the plugin files so that freeipmi plugin could be run via netdata user:    

```
# chmod u+s /usr/libexec/netdata/plugins.d/freeipmi.plugin
```
Now restart the netdata service and you could see the ipmi modules exists on dashboard.    

![/images/2020_01_03_10_51_11_165x227.jpg](/images/2020_01_03_10_51_11_165x227.jpg)

Detailed charts:    

![/images/2020_01_03_10_51_42_703x634.jpg](/images/2020_01_03_10_51_42_703x634.jpg)

and

![/images/2020_01_03_10_52_00_526x713.jpg](/images/2020_01_03_10_52_00_526x713.jpg)

