+++
title= "ltspInstallationandTrial"
date = "2024-12-03T11:20:25+08:00"
description = "ltspInstallationandTrial"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 安装ltspserver
Configuration:    

![/images/20241203_112055_x.jpg](/images/20241203_112055_x.jpg)

Installation:   

![/images/20241203_112039_x.jpg](/images/20241203_112039_x.jpg)

选择正常安装:    

![/images/20241203_112128_x.jpg](/images/20241203_112128_x.jpg)

用户名创建:    

![/images/20241203_112203_x.jpg](/images/20241203_112203_x.jpg)

配置:    

```
sudo apt update
sudo apt install -y openssh-server vim nethogs iotop
sudo add-apt-repository ppa:ltsp && sudo apt update
sudo apt install --install-recommends ltsp ltsp-binaries dnsmasq nfs-kernel-server openssh-server squashfs-tools ethtool net-tools epoptes
sudo usermod -aG epoptes test
sudo apt upgrade -y && sudo shutdown -h now
```
添加一个isolated网卡:    

![/images/20241203_113522_x.jpg](/images/20241203_113522_x.jpg)

在网络管理下，配置其ip地址为10.17.18.18：    

![/images/20241203_113929_x.jpg](/images/20241203_113929_x.jpg)


```
sudo ltsp dnsmasq --proxy-dhcp=0
sudo vim /etc/dnsmasq.d/ltsp-dnsmasq.conf
    dhcp-range=10.17.18.20,10.17.18.250,12h
sudo ltsp image /
sudo ltsp ipxe
sudo ltsp nfs
sudo ltsp initrd
sudo useradd -m test1
sudo useradd -m test2
sudo ltsp initrd
```

### 测试

![/images/20241203_114432_x.jpg](/images/20241203_114432_x.jpg)

4C 4G：    

![/images/20241203_114456_x.jpg](/images/20241203_114456_x.jpg)

命名:    

![/images/20241203_114516_x.jpg](/images/20241203_114516_x.jpg)
网络:   

![/images/20241203_114552_x.jpg](/images/20241203_114552_x.jpg)

引导:    

![/images/20241203_114613_x.jpg](/images/20241203_114613_x.jpg)

