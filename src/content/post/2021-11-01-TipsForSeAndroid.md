+++
title= "TipsForSeAndroid"
date = "2021-11-01T09:24:05+08:00"
description = "TipsForSeAndroid"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 基本概念
SEAndroid是在Android系统中基于SELinux推出的强制访问控制模型，来完善自主访问模型中只要取得root权限就可以为所欲为的情况。    
SELinux是一种基于域-类型（domain-type）模型的强制访问控制（MAC）安全系统，其原则是任何进程想在SELinux系统中干任何事，都必须先在安全策略的配置文件中赋予权限。凡是没有在安全策略中配置的权限，进程就没有该项操作的权限。在SELinux出现之前，Linux的安全模型是DAC（DiscretionaryAccess Control），译为自主访问控制。其核心思想是进程理论上所拥有的权限与运行它的用户权限相同。比如，以root用户启动shell，那么shell就有root用户的权限，在Linux系统上能干任何事。这种管理显然比较松散。在SELinux中，如果需要访问资源，系统会先进行DAC检查，不通过则访问失败，然后再进行MAC权限检查。


#### SEAndroid app分类
app分类：   

```
SELinux(或SEAndroid)将app划分为主要三种类型(根据user不同，也有其他的domain类型)：

1.untrusted_app  第三方app，没有Android平台签名，没有system权限
2.platform_app    有android平台签名，没有system权限
3.system_app      有android平台签名和system权限
4.untrusted_app_25 第三方app，没有Android平台签名，没有system权限，其定义如下This file defines the rules for untrusted apps running with targetSdkVersion <= 25.

从上面划分，权限等级，理论上：untrusted_app < platform_app < system_app按照这个进行排序
```
### app_neverallows.te
`/home/ctctest/AndroidSources/aosp11_r48/system/sepolicy/prebuilts/api/30.0/private`

Line 182, 183
Line 198, comment the `proc_version`
### private/domain.te
Changes to `/home/ctctest/AndroidSources/aosp11_r48/system/sepolicy/prebuilts/api/30.0/private/coredomain.te`
Line187, Line 113, 

### genfs_contexts
The same as in android9.0
Line43
### system_server.te
The same as in Pie
Line 987
### untrusted_app.te
The same as in Pie
Appended in last
### untrusted_app_25.te
Line 28, line 38
#### untrusted_app_27.te
Line 28, 45
#### untrusted_app_all.te
Line 161
#### webview_zygote.te
Line 66
#### zygote.te
Line 235
#### public/app.te
Line 472, Line 438
#### public/domain.te
Line 468
Line 490
Line 531
Line 970~LIne973, Line 980
Line 1348
#### public/file.te
Line 150
#### public/fsck.te
Line 21
#### public/init.te
Line 175, 181
#### public/netd.te
Line 55
#### public/platform_app.te
Append last
#### public/shell.te
Line 258, 259
#### public/system_server.te
Append last
#### public/vendor_init.te
Line 47~50
#### public/vold.te
Line 22, 23
#### public/webview_zygote.te
Append last

### Make





