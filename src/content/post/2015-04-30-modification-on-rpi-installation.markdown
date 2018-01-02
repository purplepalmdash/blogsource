---
categories: ["embedded"]
comments: true
date: 2015-04-30T00:00:00Z
title: Modification SWAP on RPI Installation
url: /2015/04/30/modification-on-rpi-installation/
---

First disable the swap partition, for it will save your sd card:    
Know where is your swap file:   

```
$ cat /proc/swap*

```
Disable it via change the S to K under `/etc/rcx.d`, and reboot the services:   

```
$ sudo mv S02dphys-swapfile K02dphys-swapfile

```
