---
categories: ["embedded"]
comments: true
date: 2014-11-08T00:00:00Z
title: Install new image to sd card for BBB
url: /2014/11/08/install-new-image-to-sd-card-for-bbb/
---

Mainly for installing the OpenWRT system on the BBB, following is the steps:    

```
# zImage
cp /media/y/embedded/BBB/svnco/trunk/bin/omap/openwrt-omap-zImage ./zImage
# dtb file
cp /media/y/embedded/BBB/svnco/trunk/bin/omap/dtbs/am335x-boneblack.dtb ./dtbs/


```

