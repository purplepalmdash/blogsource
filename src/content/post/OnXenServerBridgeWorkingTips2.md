+++
categories = ["Virtualization"]
date = "2016-08-19T14:37:59+08:00"
description = "On how to setup a virtuabox based XenServer environment"
keywords = ["Virtualization"]
title = "OnXenServerBridgeWorkingTips2"

+++
### VirtualBox Setup
Define a virtual machine:    

![/images/2016_08_19_14_36_05_517x355.jpg](/images/2016_08_19_14_36_05_517x355.jpg)    

Use 7G of 8G memory for this VM:    
![/images/2016_08_19_14_36_27_520x358.jpg](/images/2016_08_19_14_36_27_520x358.jpg)      

Create a new disk(200G), choose VDI, Dynamically allocated, And specify the location
for storing it.    

Now create the virtual machine, and configure its networking like following:     

![/images/2016_08_19_14_45_56_691x410.jpg](/images/2016_08_19_14_45_56_691x410.jpg)    

CPUs, we allocated 4:   

![/images/2016_08_19_14_48_55_532x197.jpg](/images/2016_08_19_14_48_55_532x197.jpg)     

And also the acceleration configuration:    

![/images/2016_08_19_14_50_06_380x171.jpg](/images/2016_08_19_14_50_06_380x171.jpg)   

Storage Configuration:    

![/images/2016_08_19_14_51_57_544x202.jpg](/images/2016_08_19_14_51_57_544x202.jpg)    

For saving the resources, disable USB/Audio.

Now insert the XenServer Installation CDROM, and install it.        

### XenServer Configuration
IP Address, for bridged networking:    

![/images/2016_08_19_14_56_46_486x268.jpg](/images/2016_08_19_14_56_46_486x268.jpg)     

DNS Configuration:     

![/images/2016_08_19_14_58_02_434x249.jpg](/images/2016_08_19_14_58_02_434x249.jpg)     

NTP Server Configuration:     

```
server 0.cn.pool.ntp.org
server 0.asia.pool.ntp.org
server 2.asia.pool.ntp.org
```

Now Install the XenServer.   

After installation, apply xs patches.    
