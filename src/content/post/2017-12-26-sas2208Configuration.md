+++
title = "sas2208configuration"
date = "2017-12-26T11:46:11+08:00"
description = "sas2208configuration"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Configuration Steps
Select the SAS Controller: 

![/images/2017_12_26_11_46_56_643x143.jpg](/images/2017_12_26_11_46_56_643x143.jpg)

Ignore the foreign configration, hit cancel:    

![/images/2017_12_26_11_48_36_630x157.jpg](/images/2017_12_26_11_48_36_630x157.jpg)

Display all of the disks:    

![/images/2017_12_26_11_49_30_636x411.jpg](/images/2017_12_26_11_49_30_636x411.jpg)

Select one disk(1T-Disk):    

![/images/2017_12_26_11_51_50_630x436.jpg](/images/2017_12_26_11_51_50_630x436.jpg)

Click configuration wizard:    

![/images/2017_12_26_11_55_26_634x372.jpg](/images/2017_12_26_11_55_26_634x372.jpg)

Select Configuration Method:    

![/images/2017_12_26_11_56_26_633x290.jpg](/images/2017_12_26_11_56_26_633x290.jpg)

Configure a new disk:    

![/images/2017_12_26_11_59_41_629x344.jpg](/images/2017_12_26_11_59_41_629x344.jpg)

Click "Accept DG" for accepting the configurations.   

![/images/2017_12_26_12_00_42_633x354.jpg](/images/2017_12_26_12_00_42_633x354.jpg)

Ignore add Span:    

![/images/2017_12_26_12_02_42_638x362.jpg](/images/2017_12_26_12_02_42_638x362.jpg)


Add to Span:    

![/images/2017_12_26_12_04_03_633x341.jpg](/images/2017_12_26_12_04_03_633x341.jpg)

![/images/2017_12_26_12_05_45_635x418.jpg](/images/2017_12_26_12_05_45_635x418.jpg)


![/images/2017_12_26_12_06_39_634x435.jpg](/images/2017_12_26_12_06_39_634x435.jpg)


Now you add successfully.  

Click Accept:    

![/images/2017_12_26_12_07_17_627x317.jpg](/images/2017_12_26_12_07_17_627x317.jpg)

Click Save Configuration, and you will be asked to initialize the new virtual
driver, click yes.    

![/images/2017_12_26_12_08_48_567x190.jpg](/images/2017_12_26_12_08_48_567x190.jpg)


Then save and reboot your system, your configuration will be done.    

### System
Install a new system under /dev/sde, then you could use following command for
setting the grub configuration:    

```
# os-prober
# grub2-mkconfig -o /boot/grub2/grub.cfg
``` 
Then you could get your newly installed system working.    
