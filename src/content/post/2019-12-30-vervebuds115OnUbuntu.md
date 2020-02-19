+++
title = "VerveBuds115OnUbuntu"
date = "2019-12-30T09:19:48+08:00"
description = "VerveBuds115OnUbuntu"
keywords = ["Bluetooth"]
categories = ["Technology"]
+++
Ubuntu18.04 configurating bluetooth headset VerveBuds115 steps:    

### Blueman
Use blueman for configrating the headset connection/configuration.    

```
# sudo apt-get install -y blueman
```
Add blueman into awesome's startup function:     

```
# cat ~/.config/awesome/rc.lua | grep blueman
run_once("blueman-applet &")
```
Configurating the blueman:    

![/images/2019_12_30_09_22_31_969x154.jpg](/images/2019_12_30_09_22_31_969x154.jpg)

Choose A2dp.    

### Sound
Ubuntu18.04 use pulseaudio for default sound backend, so we use following tools for configurating the sound:     

```
# sudo apt-get install -y pasystray
# sudo apt-get install -y pnmixer
```
Also add them into the awesome's startup functions, thua after system bootup we could find the volume controlling in systray:    

```
# vim ~/.config/awesome/rc.lua
.....
run_once("blueman-applet &")
run_once("pnmixer &")
run_once("pasystray &")
```
Bugs:  By pnmixer we could only  controlling the Intel PCH, but not bluetooth?    

