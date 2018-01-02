+++
categories = ["Linux"]
keywords = ["Linux"]
date = "2017-07-04T16:28:35+08:00"
description = "Make docker images offline"
title = "MakeDockerImages"

+++
### 环境准备
在Virtualbox（已默认安装在虚拟机上）中，创建一台新机器，如下图：   

![/images/2017_07_04_14_48_51_534x451.jpg](/images/2017_07_04_14_48_51_534x451.jpg)

内存为1G:    

![/images/2017_07_04_14_49_06_528x362.jpg](/images/2017_07_04_14_49_06_528x362.jpg)

创建硬盘:    

![/images/2017_07_04_14_49_18_524x363.jpg](/images/2017_07_04_14_49_18_524x363.jpg)

选择VDI:  

![/images/2017_07_04_14_49_38_437x201.jpg](/images/2017_07_04_14_49_38_437x201.jpg)

Dynamically Allocated:    

![/images/2017_07_04_14_50_00_426x271.jpg](/images/2017_07_04_14_50_00_426x271.jpg)

大小为8G：    

![/images/2017_07_04_14_50_12_440x255.jpg](/images/2017_07_04_14_50_12_440x255.jpg)

现在点击Settings, 加载安装盘:    

![/images/2017_07_04_14_51_17_698x419.jpg](/images/2017_07_04_14_51_17_698x419.jpg)

点击OK后，按Start开始安装：    

选择`Basic Server`，   

![/images/2017_07_04_14_53_39_559x274.jpg](/images/2017_07_04_14_53_39_559x274.jpg)

安装完毕后重新启动系统。    

点击virtualbox菜单上的`Devices->Optical Devices->`
，加载rhel6.6的ISO到虚拟机。    

通过下列命令集合，创建rhel6.6的基础文件系统:    

```
# mkdir -p /mnt/rhel6-repo
# mount /dev/sr0 /mnt/rhel6-repo
# mkdir /root/rhel6-root
# rpm --root /root/rhel6-root/ --initdb
# rpm --root /root/rhel6-root/ -ivh /mnt/rhel6-repo/Packages/redhat-release-server-6Server-6.6.0.2.el6.x86_64.rpm 
# cd /root/rhel6-root/
# cd etc/yum.repos.d
# rm -f *.repo
# vim rhel6.repo
[rhel6]
baseurl=file:///mnt/rhel6-repo
enabled=1
gpgcheck=0
# rpm --root /root/rhel6-root --import /mnt/rhel6-repo/RPM-GPG-KEY-redhat-*
# yum -y --installroot=/root/rhel6-root install yum which vim

```
这时候如果直接打包成`tar.gz`文件，我们则拥有了一个全新的docker
rhel6.6文件根分区系统:     

```
# tar czvf rhel6-root.tar.gz rhel6-root/
# scp ./rhel6-root.tar.gz root@xxx.xxx.xxx.xxx:/home/xxx
```

我们可以手动更改repo的位置，因为一旦解压开，最好从某个http服务器获得包,
而这个服务器上的目录则是原封不动的拷贝所有的ISO文件里的内容:    

```
# vim etc/yum.repos.d/rhel6.repo 
[rhel6]
baseurl=http://192.168.0.220/rhel
enabled=1
gpgcheck=0
```

在已经安装了Docker的机器上，可以通过如下命令来添加Docker镜像.    

```
$ sudo tar -C rhel6-root/ -c . | sudo docker import - myrhel66
```

运行制作好的Docker镜像:    

```
$  sudo docker run -it myrhel66 /bin/bash
# yum clean all && yum makecache 
```

### 桌面化环境

```
# yum distribution-synchronization
# 
```


