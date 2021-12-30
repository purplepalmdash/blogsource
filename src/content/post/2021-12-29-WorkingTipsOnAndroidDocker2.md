+++
title= "WorkingTipsOnAndroidDocker2"
date = "2021-12-29T11:56:15+08:00"
description = "WorkingTipsOnAndroidDocker2"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 目的
记录aosp avd 11上启动容器的操作事项.    

### 步骤
因为内核经过修改，故启动的时候需要点击`ok`后进入系统:    

![/images/2021_12_29_11_56_51_356x415.jpg](/images/2021_12_29_11_56_51_356x415.jpg)

确保wifi连接:   

![/images/2021_12_29_11_58_07_347x482.jpg](/images/2021_12_29_11_58_07_347x482.jpg)

进入到adb shell:   

```
$ adb root
restarting adbd as root
$ adb shell
generic_x86_64:/ # ip addr
```

添加路由规则:    

```
ip rule add pref 1 from all lookup main
ip rule add pref 2 from all lookup default
ip route add default via 192.168.91.254 dev wlan0
ip rule add from all lookup main pref 30000
```
启动dockerd进程:     

```
dockerd  --dns=223.5.5.5 --data-root=/data/var/ --ip=192.168.89.153 & >/data/dockerd-logfile 2>&1
```
启动docker:    

```
# docker run -d --privileged -p 8888:5555 redroid/redroid:8.1.0-latest
# docker run -d --privileged -p 8888:5555 redroid/redroid:9.0.0-latest
# docker run -d --privileged -p 8888:5555 redroid/redroid:10.0.0-latest
```
