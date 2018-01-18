+++
title = "WorkingTipsOnk8sLVM"
date = "2018-01-16T10:31:14+08:00"
description = "Workingtipsonk8slvm"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Preparation
Define ip address/gateway:    

![/images/2018_01_16_10_31_58_494x339.jpg](/images/2018_01_16_10_31_58_494x339.jpg)

Set hostname: 

![/images/2018_01_16_10_32_18_439x180.jpg](/images/2018_01_16_10_32_18_439x180.jpg)

Disk Preparation:    

![/images/2018_01_16_10_36_09_758x404.jpg](/images/2018_01_16_10_36_09_758x404.jpg)

### Configuration
In kismatic, configure like following:    

![/images/2018_01_16_10_39_10_819x493.jpg](/images/2018_01_16_10_39_10_819x493.jpg)

Then install kismatic as usual.   
### helm
When in offline-mode, `helm repo update` will failed.   

Aim: install monocular in cluster.     
