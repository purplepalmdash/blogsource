+++
title= "WorkingTipsOnRPI4Android"
date = "2021-08-10T11:59:24+08:00"
description = "WorkingTipsOnRPI4Android"
keywords = ["Technology"]
categories = ["Technology"]
+++
### WorkingSteps
Operation steps:   

```
# adb connect 192.168.1.113
already connected to 192.168.1.113:5555
# adb shell
rpi4:/ 
```
Or:   

```
# adb root
restarting adbd as root
# adb shell
rpi4:/ # ls
acct boot       charger data          dev  init.environ.rc      init.usb.rc      mnt proc             res    sepolicy system     vendor_service_contexts 
apex bugreports config  debug_ramdisk etc  init.rc              init.zygote32.rc odm product          sbin   storage  ueventd.rc 
bin  cache      d       default.prop  init init.usb.configfs.rc lost+found       oem product_services sdcard sys      vendor     
rpi4:/ # exit
```
OR:   

```
adb remount
remount succeeded
➜  Downloads adb shell  
rpi4:/ # cd /system/bin/                                                                                                                                                                     
rpi4:/system/bin # touch fff
rpi4:/system/bin # rm fff                                                                                                                                                                    
rpi4:/system/bin #                                                                           


```

Shutdown the phone:    

```
sudo adb shell reboot -p
```
