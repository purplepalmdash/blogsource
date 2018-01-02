+++
title = "WTwareOnRaspberryPI"
date = "2017-11-28T14:54:34+08:00"
description = "WTwareOnRaspberryPI"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 效果

![/images/2017_11_28_15_06_56_485x642.jpg](/images/2017_11_28_15_06_56_485x642.jpg)

![/images/2017_11_28_15_07_19_371x496.jpg](/images/2017_11_28_15_07_19_371x496.jpg)

![/images/2017_11_28_15_07_39_378x348.jpg](/images/2017_11_28_15_07_39_378x348.jpg)

### 背景
RaspberryPI上的一种瘦客户端解决方案。
### 步骤
下载地址：    

[https://winterminal.com/index.html#download](https://winterminal.com/index.html#download)    

下载链接有两个，一个是exe下载，一个是zip下载，我们选择exe下载。    

![/images/2017_11_28_14_56_35_346x230.jpg](/images/2017_11_28_14_56_35_346x230.jpg)
下载包大小大概200M不到，安装在Windows系统上，附带的例如TFTP/dhcpD之类的功能根据个人需求自行选择。我这里因为是静态地址的，不需要这些组件，就没有安装。    

安装完毕后的配置过程如下，可以看到我们

![](/images/2017_11_28_09_10_28_1053x783.jpg)

![](/images/2017_11_28_09_11_33_846x757.jpg)

![](/images/2017_11_28_09_12_57_847x761.jpg)

![](/images/2017_11_28_09_15_18_860x761.jpg)

![](/images/2017_11_28_09_15_38_851x768.jpg)

这类说明一下，我们先配置RaspberryPI本身的IP地址/子网掩码/网关等信息。远端的服务器则在开机启动的时候再配置。最后填入的密码是进入设置页面的密码。在开机加电时按DEL键进入修改的条目。    

### 开机启动
这里的配置过程以后再补上，因为当时没有截图。    

所有的更改项对应在SD卡的Config目录下。    

```
$  cat initrd.wtc 
clientIP = 192.xxx.xxx.xxx
netmask = 255.255.255.0
routerIP = 192.xxx.xxx.xxx
nameserverIP = 192.xxx.xxx.xxx
config = local
setupPassword = xxx.xxxxxxxxxxxxxxx
$ cat config.wtc 
server = rdp:192.xxx.xxx.xxx
User=username:password
```

配置好以后，插入SD卡就可以进入到我们预配置好的远程桌面了，这里我们启用的是RDP协议的Windows远程桌面。    

为了保证服务质量，WTware默认禁掉了很多桌面效果，可以通过在`config.wtc`文件中加入下列条目，来开启所有特效:    

```
graphic = abcdefg
```
