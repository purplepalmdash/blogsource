+++
title = "TipsOnKismaticTransfer"
date = "2018-01-15T11:46:05+08:00"
description = "TipsOnKismaticTransfer"
keywords = ["Linux"]
categories = ["Linux"]
+++
主要记录生产环境的迁移，后期可以作为实际部署和安装时的文档参考。     

### 网络准备
创建一个`10.15.205.0/24`网络:    

![/images/2018_01_15_11_46_51_334x401.jpg](/images/2018_01_15_11_46_51_334x401.jpg)
由于在内网，故指定NAT与否都无所谓,
在有Internet的环境中，可以指定为隔离环境，以模拟实际的现网环境:    

![/images/2018_01_15_11_47_47_436x322.jpg](/images/2018_01_15_11_47_47_436x322.jpg)
### 镜像准备
直接将镜像拷贝到内网，建立aptly虚拟机、registry
mirror虚拟机即可。未来将对aptly虚拟机和registry虚拟机做合并操作。    

基础镜像准备, 使用Ubuntu1604_base.qcow2镜像文件，做以下操作:    

```
0. qemu-system-x86_64 -net nic -net user,hostfwd=tcp::2288-:22 -hda xxxx.qcow2
   -m 1024 --enable-kvm
1. root密码修改
2. root远程登录(/etc/ssh/sshd_config)
3. /etc/hosts中添加`10.15.205.2	mirror.xxxx.com`条目，以使用registry mirror.
4. 将免密码登录的公钥插入到操作系统中 
5. 配置aptly的镜像服务器
vim /etc/apt/sources.list
deb http://10.15.205.3 xenial main
deb http://10.15.205.3 kubernetes-xenial main
deb [arch=amd64] http://10.15.205.3 xenial stable
6. 导入aptly所需的签名
cat mykey.asc | apt-key add -
```
做完此操作后，保存基础镜像，创建出来相应的虚拟机。   

![/images/2018_01_15_14_55_37_1058x179.jpg](/images/2018_01_15_14_55_37_1058x179.jpg)



