+++
title = "TipsOnKubespray28Upgrading"
date = "2018-12-11T09:08:00+08:00"
description = "TipsOnKubespray28Upgrading"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 初始化配置
按照AI组的设想，基于Ubuntu16.04来做，后续其实也可以基于Ubuntu16.05来做，应该是一样的。    
创建一个虚拟机，192.168.122.177/24, 安装好redsocks.    

visudo(nopasswd), `apt-get install -y nethogs build-essential libevent-dev`.   

做成基础镜像后，关机，undefine此虚拟机。   


```
# 
``` 
