+++
date = "2017-02-14T11:08:07+08:00"
categories = ["Linux"]
keywords = ["Encryption"]
description = "Encrypt whole disk of OpenSUSE"
title = "OpenSuSE全盘加密"

+++
### 系统准备
OpenSuse 13.2 DVD, Libvirt.    

### 磁盘分区
在初始化磁盘配置的时候，点击`Edit Proposal Settings`:   
![/images/2017_02_14_11_09_56_571x399.jpg](/images/2017_02_14_11_09_56_571x399.jpg)
点击`Create LVM-based Proposal`并勾选`Encrypt Volume Group`:    
![/images/2017_02_14_11_10_52_247x341.jpg](/images/2017_02_14_11_10_52_247x341.jpg)
输入两次密码：    
![/images/2017_02_14_11_11_41_258x351.jpg](/images/2017_02_14_11_11_41_258x351.jpg)
可以选择`ext4`为默认的文件系统:    
![/images/2017_02_14_11_12_09_254x343.jpg](/images/2017_02_14_11_12_09_254x343.jpg)
磁盘分区格局如下:    
![/images/2017_02_14_11_12_50_387x101.jpg](/images/2017_02_14_11_12_50_387x101.jpg)
点击`Next`按钮，继续完成安装.    

安装完毕后，需要手动输入密码才能进入系统:    
![/images/2017_02_14_11_12_50_387x101.jpg](/images/2017_02_14_11_12_50_387x101.jpg)

### 免密码登录
OpenSuse的免密码登录与CentOS相同, 都是通过修改dracut来实现免密码登录.    
