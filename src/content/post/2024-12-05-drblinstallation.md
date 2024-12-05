+++
title= "drblinstallation"
date = "2024-12-05T16:19:02+08:00"
description = "drblinstallation"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 安装
先试用，配置好网络后，安装:    

![/images/20241205_172406_x.jpg](/images/20241205_172406_x.jpg)


![/images/20241205_161910_x.jpg](/images/20241205_161910_x.jpg)

进入系统后的安装和配置:    

```
sudo apt update -y && sudo apt upgrade -y
sudo wget -O /etc/apt/trusted.gpg.d/drbl-gpg.asc https://drbl.org/GPG-KEY-DRBL
sudo vim /etc/apt/sources.list
    deb http://free.nchc.org.tw/ubuntu jammy main restricted universe multiverse
    deb http://free.nchc.org.tw/drbl-core drbl stable
sudo apt update
sudo apt install -y drbl
```
配置:    

```
ufw disable
sudo drblsrv -i
```

![/images/20241205_164835_x.jpg](/images/20241205_164835_x.jpg)

![/images/20241205_164853_x.jpg](/images/20241205_164853_x.jpg)

![/images/20241205_164934_x.jpg](/images/20241205_164934_x.jpg)

