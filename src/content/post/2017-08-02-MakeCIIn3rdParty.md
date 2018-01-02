+++
title = "MakeCIIn3rdParty"
date = "2017-08-02T15:00:52+08:00"
description = "MakeCIIn3rdParty"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Docker Images
需要用到的Docker Image: `wgetbuildcs6`, 构建的Dockerfile:    

```
FROM centos:centos6

MAINTAINER dash xxx <xxxx@gmail.com>

RUN yum -y install curl git gcc make rpm-build python-devel which lrzsz tar gnutls gnutls-devel
```
将创建好的镜像上传到私有仓库(某台内网主机):    

```
$ sudo docker load<wgetbuildcs6.tar
$ sudo docker images | grep wgetbuildcs6
$ sudo docker tag 1020xxxxx 192.168.124.102:5000/xxxxx/wgetbuildcs6:latest
$ sudo docker push  192.168.124.102:5000/xxxxx/wgetbuildcs6:latest
```

![/images/2017_08_02_15_12_12_484x294.jpg](/images/2017_08_02_15_12_12_484x294.jpg)

### Git Repository
在CentOS7.3系统上，安装git daemon:    

```
# yum install -y git-daemon
```

在源码目录下, 执行以下命令:    

```
# git init
# git add .
# git commit -m "initial commit"
# cd ..
# git clone --bare wgetbuild wgetbuild.git
```

![/images/2017_08_02_15_43_19_426x427.jpg](/images/2017_08_02_15_43_19_426x427.jpg)

现在运行:    

```
# git daemon --verbose --export-all --base-path=.
```
即可激活git server.  
测试:    

```
# git clone git://192.192.192.91/wgetbuild.git mybuild
``` 

证明友商方案对git协议支持不佳，切换回gitlab，用http继续测试。     


### Enable push in git
Enable push to git server in git daemon:   

```
# git daemon --enable=receive-pack --verbose --export-all --base-path=.
```
