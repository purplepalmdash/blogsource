+++
title= "NoVNCOperationOn4ANetwork"
date = "2022-12-26T21:11:05+08:00"
description = "NoVNCOperationOn4ANetwork"
keywords = ["Technology"]
categories = ["Technology"]
+++
因4A速度实在太慢，通过bmc远程安装系统基本上不可行。故采用在跳板机上启动novnc加持的桌面session的方式用来部署操作系统。

登陆跳板机`192.168.xx.xx`后，运行以下命令建立一个监听跳板机6080端口的`lxde`桌面环境（因跳板机是arm64环境，需要指定arm64后缀的镜像）:    

```
sudo docker run -p 6080:80 -e USER=doro -e PASSWORD=password -v /dev/shm:/dev/shm dorowu/ubuntu-desktop-lxde-vnc:focal-arm64
```

通过xshell和proxifier建立全局代理后，在浏览器里直接访问`192.168.xx.xx:6080`即可访问到创建的桌面环境:    

![/images/2022_12_26_21_21_39_1048x730.jpg](/images/2022_12_26_21_21_39_1048x730.jpg)

在该桌面环境里可直接调用firefox下载相关镜像，并使用bmc挂载该镜像用于安装操作系统:     

![/images/2022_12_26_21_24_22_1240x783.jpg](/images/2022_12_26_21_24_22_1240x783.jpg)

因为安装介质都在同一数据中心内，所以安装时不会出明显卡顿，大约15～30分钟内即可安装成功。   

安装完后的bmc界面:    

![/images/2022_12_26_21_23_41_1018x495.jpg](/images/2022_12_26_21_23_41_1018x495.jpg)


