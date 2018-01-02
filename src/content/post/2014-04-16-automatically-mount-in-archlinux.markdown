---
categories: ["Linux"]
comments: true
date: 2014-04-16T00:00:00Z
title: Automatically Mount in ArchLinux
url: /2014/04/16/automatically-mount-in-archlinux/
---

Use udisk/udisk2/udiskie for automatically mount usb disks.     

```
	pacman -S udisk udisk2 udiskie

```
Add following line into the .xinitrc:    

```
	udiskie -2 --tray &

```
If you want to umount all media with the command:

```
	udiskie-umount -a

```
umount speicified disk partition:

```
	udiskie-umount /media/MY_USB_DRIVE

```
