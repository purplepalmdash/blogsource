+++
keywords = ["Linux"]
date = "2017-02-04T20:14:50+08:00"
description = "Enable USB camera on ArchLinux"
title = "EnableUSBCameraOnArchLinux"
categories = ["Linux"]

+++
### WebCam Detection
Detect the USB equipment via:    

```
# lsusb
Bus 002 Device 002: ID 0b95:772b ASIX Electronics Corp. AX88772B
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 008 Device 002: ID 0c45:0011 Microdia EBUDDY
Bus 008 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 007 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 006 Device 001: ID 1d6b:0001 Linux Foundation 1.1 root hub
Bus 001 Device 002: ID 046d:0829 Logitech, Inc. 
.....
```
Then we got more informations from our detected webCAM equipment:    

```
# lsusb -s 001:002 -v | egrep "Width|Height"
        wWidth                            640
        wHeight                           480
        wWidth                            352
        wHeight                           288
.....
```

### Software
Install guvcview for detection the cameral and take pictures:    

```
# pacman -S guvcview
```
Now using `guvcview` you could get the pictures.     

