+++
title = "PlayWithDockerUsage"
date = "2018-08-14T15:36:48+08:00"
description = "PlayWithDockerUsage"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 环境准备
推荐配置：    
两核以上虚拟机或物理机    
4G以上内存(推荐需要8G以上)    
40G以上磁盘空间    
网卡x1.    
### 系统安装
使用光盘启动系统，`ubuntu-18.04.1-2018.08.14-playdocker-amd64.iso`光盘里含有操作系统及运行PlayWithDocker所需的全部离线包。将此光盘刻录或使用U盘加载物理机/虚拟机，启动系统至安装界面:     

![/images/2018_08_14_15_41_41_875x505.jpg](/images/2018_08_14_15_41_41_875x505.jpg)

选择`Install Ubuntu`，进入到下一步：    

![/images/2018_08_14_15_42_20_939x362.jpg](/images/2018_08_14_15_42_20_939x362.jpg)

键盘布局默认不变，点击`Continue`，进入到下一步：    

![/images/2018_08_14_15_43_02_660x231.jpg](/images/2018_08_14_15_43_02_660x231.jpg)

直接点击`Continue`到下一步:    

![/images/2018_08_14_15_43_35_700x297.jpg](/images/2018_08_14_15_43_35_700x297.jpg)

选择`Erase disk and install Ubuntu`后，点击`Install Now`进入到下一步:    

![/images/2018_08_14_15_44_17_870x217.jpg](/images/2018_08_14_15_44_17_870x217.jpg)

弹出的对话框中点击`Continue`，写入磁盘修改.    

![/images/2018_08_14_15_44_52_941x414.jpg](/images/2018_08_14_15_44_52_941x414.jpg)

时区随便填，点击`Continue`到下一步:    

![/images/2018_08_14_15_45_40_930x482.jpg](/images/2018_08_14_15_45_40_930x482.jpg)

填入用户名/密码配置信息后，点击`Continue`进入到系统安装阶段。    

系统安装需要大约10～20分钟，具体时间取决于你的硬件配置。全程无需干预。    

![/images/2018_08_14_16_00_24_738x149.jpg](/images/2018_08_14_16_00_24_738x149.jpg)

点击`Restart Now`，卸载ISO后重启机器，系统安装完成。    

### Play With Docker
安装完毕后，登录系统:    

![/images/2018_08_14_16_04_19_448x275.jpg](/images/2018_08_14_16_04_19_448x275.jpg)
运行`terminal`， 打开终端控制器：    

![/images/2018_08_14_16_05_14_378x243.jpg](/images/2018_08_14_16_05_14_378x243.jpg)

运行以下命令:    

```
test@testPC:~$ sudo bash
[sudo] password for test:
root@testPC:~# cd /root
root@testPC:/root# ./initial.sh
```
运行完毕后可以通过`docker ps`检查所需服务的启动情况。    

外部访问机器的4000端口，可以看到PlayWithDocker已正常运行,
推荐使用新版chrome浏览器访问:    

![/images/2018_08_14_16_14_16_679x649.jpg](/images/2018_08_14_16_14_16_679x649.jpg)

### Play With Kubernetes
安装流程和上述流程相似。使用的ISO名称为`ubuntu-18.04.1-2018.07.27.pwd-desktop.amd64.iso`.    

![/images/2018_08_14_17_44_48_621x602.jpg](/images/2018_08_14_17_44_48_621x602.jpg)
   
