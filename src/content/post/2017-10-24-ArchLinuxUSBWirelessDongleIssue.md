+++
title = "ArchLinuxUSBWirelessDongleIsssue"
date = "2017-10-24T08:53:47+08:00"
description = "ArchLinuxUSBWirelessDongleIsssue"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Hardware & Software
Tenda u1, rtl8192eu chipset.    
Could be found as wireless adaptor, but could not get networking.    

```
# sudo wifi-menu wlp37s0u1
# dmesg
....
[42647.461034] wlp37s0u1: send auth to **:xx:xx:xx:xx:xx (try 1/3)
[42647.663969] wlp37s0u1: send auth to **:xx:xx:xx:xx:xx (try 2/3)
[42647.867339] wlp37s0u1: send auth to **:xx:xx:xx:xx:xx (try 3/3)
[42648.070657] wlp37s0u1: authentication with xx:xx:xx:xx:xx:xx timed out
```
Kernel Version:    

```
$ uname -r
4.13.7-1-ARCH
```

### Trouble-Shooting
Install `wireless_tools` and use `iwconfig` for detecting the hardware:   

```
$ sudo pacman -S wireless_tools
$ sudo iwconfig
....
wlp37s0u1  IEEE 802.11  ESSID:off/any  
          Mode:Managed  Access Point: Not-Associated   Tx-Power=20 dBm   
          Retry short limit:7   RTS thr=2347 B   Fragment thr:off
          Encryption key:off
          Power Management:off

```
Seems the driver have some problem, so I got the following url, and run the
same procedures as its comments:    

[https://aur.archlinux.org/packages/8192eu-dkms/](https://aur.archlinux.org/packages/8192eu-dkms/)   

### 8192eu-dkms
Install it via:    

```
$ yaourt 8192eu-dkms
``` 
Configure it via:    

```
$ sudo vim /etc/modprobe.d/wlan.conf
blacklist rtl8xxxu
install rtl8xxxu /bin/false
```
Now reboot machine and examine the `rtl8xxxu` is not loaded via `lsmod | grep
-i 8xx`, since we use the right device driver, our wireless adapter will work
properly and connect to wifi ap.   
