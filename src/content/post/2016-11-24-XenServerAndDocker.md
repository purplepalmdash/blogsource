+++
categories = ["Virtualization"]
date = "2016-11-24T09:44:04+08:00"
description = "XenServer and Docker interactive"
keywords = ["Virtualization"]
title = "XenServerAndDocker"

+++
### XenServer配置
#### XenServer 6.5
下载xscontainer iso并安装之:     

```
# wget http://downloadns.citrix.com.edgesuite.net/10343/XenServer-6.5.0-SP1-xscontainer.iso
# xe-install-supplemental-pack XenServer-6.5.0-SP1-xscontainer.iso
```
#### XenServer 7.0
下载并安装:    

```
# wget http://downloadns.citrix.com.edgesuite.net/11621/XenServer-7.0.0-xscontainer.iso
# xe-install-supplemental-pack XenServer-7.0.0-xscontainer.iso
```

### guest虚拟机准备
安装docker:   

```
$ curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/experimental/internet | sh
$ sudo usermod -aG docker 
```
安装ncat/openssh-server(nmap中包含ncat):    

```
$ sudo apt-get install -y openssh-server nmap
```
### 添加guest虚拟机
添加docker monitor:    

```
# xscontainer-prepare-vm -v 05cd5c8f-eb32-86c6-b687-7a296180e3d3 -u dash
```
添加后的效果如下:    
