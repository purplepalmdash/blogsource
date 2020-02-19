+++
title = "MigratingBuildISOVM"
date = "2018-06-27T14:35:28+08:00"
description = "MigratingBuildISOVM"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Background
环境迁移，BuildISO的虚拟机IP更换时，需要进行的配置。如，从192.168.122.0/24到192.168.124.0/24网络环境时需要做的修改.    
具备的文件:  makecentosiso.qcow2, 原有配置, IP为192.168.122.222/24,
迁移为192.168.124.200/24.    
### VM
方法一： 配置一个192.168.122.0/24的网段， virt-manager中新建一个即可。   

方法二： 如下步骤:    

配置虚拟机，8192MB/4CPUs, `zz_makecentosisso`.   

nmtui, 配置网卡:    

![/images/2018_06_27_14_50_31_606x349.jpg](/images/2018_06_27_14_50_31_606x349.jpg)

进入系统后，配置gitlab的external url地址:    

```
# vim /etc/gitlab/gitlab.rb
......

external_url ' http://192.168.124.200'
.....

```
重新启动gitlab服务:    

```
# gitlab-ctl reconfigure && gitlab-ctl restart
```
重新启动服务应该可以看到gitlab界面(用户名/密码为root/thinker@1):    

![/images/2018_06_27_15_01_29_530x360.jpg](/images/2018_06_27_15_01_29_530x360.jpg)

gitlab runner也需要重新配置，之前配置为192.168.122.222->192.168.124.200,
检查方式在: settings->CI/CD, 可以看到runner为灰色,
这是因为IP地址没有更改的缘故:    

![/images/2018_06_27_15_05_03_456x184.jpg](/images/2018_06_27_15_05_03_456x184.jpg)

更新方式:    

```
# vim /etc/gitlab-runner/config.toml

url= "http://192.168.124.200"
```
一般情况下，现在可以继续在gitlab里进行CI/CD， 如果出现异常，则需要在admin
area下重新激活一下runner.   

