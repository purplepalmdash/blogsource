---
categories: ["Linux", "Bluetooth"]
comments: true
date: 2013-12-14T00:00:00Z
title: Bluetooth headset in ArchLinux
url: /2013/12/14/bluetooth-headset-in-archlinux/
---

I have a Nokia bluetooth headset BH-105,and a bluetooth usb dongle. And I want to connect them together. Following are the steps.     
###Software Preparation
On ArchLinux, Install "bluez" and "bluez-utils"

```
	$ sudo pacman -S bluez bluez-utils
	$ sudo systemctl start bluetooth && sudo systemctl enable bluetooth

```

