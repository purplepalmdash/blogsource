+++
title= "WorkingTipsForXenVmExport"
date = "2024-02-22T19:12:36+08:00"
description = "WorkingTipsForXenVmExport"
keywords = ["Technology"]
categories = ["Technology"]
+++
在XenServer虚机实例中，删除`1`中所示意的两个软件， 将`C:\windows\system32\drivers\xen.sys`删除或者转移目录:    

![/images/2024_02_22_19_15_34_749x514.jpg](/images/2024_02_22_19_15_34_749x514.jpg)

从Xencenter导出镜像文件, 可通过XenCenter或者命令行导出，导出格式为xva格式.     

编译xva-img：    

```
$ git clone https://github.com/eriklax/xva-img.git
$ cd xva-img/
$ sudo apt install -y cmake libxxhash-dev
$ cmake .
$ sudo make install
$ which xva-img
/usr/local/bin/xva-img
```
解压xva文件并进行转换:     

```
$ mkdir win10 && cd win10 && tar xf ../20240222XXXXXXXXXXX.xva
$ cd ..
$ xva-img -p disk-export win10/Ref\:21/ virtual_win10.raw
```
使用导出的`virtual_win10.raw`文件启动Qemu虚机, 注意磁盘类型为SATA:    

![/images/2024_02_22_19_22_08_481x314.jpg](/images/2024_02_22_19_22_08_481x314.jpg)


