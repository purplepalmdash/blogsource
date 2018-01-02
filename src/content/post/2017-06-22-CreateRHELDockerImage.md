+++
date = "2017-06-22T16:22:37+08:00"
categories = ["Linux"]
keywords = ["Docker"]
description = "Create rhel docker image offline"
title = "CreateRHEL6DockerImage"

+++
### 背景
内网工作环境下，无法联网安装软件，所以需要制作rhel6的docker镜像，用于离线安装、验证、部署等过程。    

### 准备
需要准备一台基于virtualbox的rhel65虚拟机。    
一台安装了docker的Centos7或者ubuntu的物理机。    
rhel65安装光盘.    

### 步骤
启动rhel65虚拟机，将rhel65的iso挂载到virtualbox虚拟机上.   
登录到虚拟机后，执行以下命令:    

```
# mkdir -p /mnt/rhel6-repo
# mount /dev/sr0 /mnt/rhel6-repo
# mkdir /root/rhel6-root
# rpm --root /root/rhel6-root/ --initdb
# rpm --root /root/rhel6-root/ -ivh /mnt/rhel6-repo/Packages/redhat-release-server-6Server-6.5.0.1.el6.x86_64.rpm
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
安装完毕后，将`/root/rhel6-root`目录拷贝到安装好docker的物理机，执行以下命令打包成docker镜像:    

```
# tar -C rhel6-root/ -c . | docker import - rhel65
# docker images | grep rhel65
```

现在你拥有了自己的rhel65镜像，可以直接运行之。    
