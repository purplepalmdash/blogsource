+++
title= "dvChangelog"
date = "2023-10-19T15:26:59+08:00"
description = "dvChangelog"
keywords = ["Technology"]
categories = ["Technology"]
+++
Changed `DVServer/Driver.h`:    

```
Line 25: 
//#define DVSERVER_HWDCURSOR
-->
#define DVSERVER_HWDCURSOR

Line 45->Line 58, CursorData, all commented
```
Compilation will give following errors `'device_iface_data':undeclared identifier`:      

![/images/2023_10_19_15_30_50_775x519.jpg](/images/2023_10_19_15_30_50_775x519.jpg)

Comment `DVServer/Driver.cpp` Line 543 -> Line 547, rebuild. 
