+++
title = "MakingOfflineKubeSprayDeploymentISO"
date = "2018-11-15T11:25:30+08:00"
description = "MakingOfflineKubeSprayDeploymentISO"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 1. Portus纯净版制作
vagrant启动ubuntu14.04， 安装docker/docker-compose, 注意事项:    

```
$ sudo apt-get purge lxc-docker-1.9.0
$ sudo apt-get install \
    linux-image-extra-$(uname -r) \
    linux-image-extra-virtual
$ sudo apt-get update
$ sudo apt-get install -y \
    apt-transport-https \
        ca-certificates \
            curl \
                software-properties-common

$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) \
         stable"
$ sudo apt-get update
$ sudo apt-get install -y docker-ce
$ sudo apt-get install -y libyaml-dev libpython-dev
$ sudo pip uninstall docker-py
$ sudo pip uninstall docker-compose
$ sudo pip install --upgrade --force-reinstall  docker-compose
```
而后用我们预先定义好的compose目录(compose.tar.gz解压到/usr/local/compose目录下), 更改IP地址与域名的映射，`extra_hosts:`条目, 
全翻墙条件下，一条命令启动:    

```
# cd /usr/local/compose
# docker-compose up
```
因为我们预先已经定义好了域名与IP的配置，我们这里定义的内网IP为`192.168.33.18`,
于是在有浏览器的`192.168.33.1`机器上，配置`/etc/hosts`下的DNS条目：    

```
# vim /etc/hosts
portus.xxxx.com	192.168.33.18
```
打开浏览器访问`https://portus.xxxx.com`，第一次登录需要配置admin的邮箱及密码:    

![/images/2018_11_15_11_34_49_515x420.jpg](/images/2018_11_15_11_34_49_515x420.jpg)

配置远端registry仓库:    

![/images/2018_11_15_11_37_44_379x197.jpg](/images/2018_11_15_11_37_44_379x197.jpg)

创建一个`kubespray`团队:    

![/images/2018_11_15_11_39_26_913x404.jpg](/images/2018_11_15_11_39_26_913x404.jpg)

User条目中，创建一个kubespray的用户:    

![/images/2018_11_15_11_41_24_477x321.jpg](/images/2018_11_15_11_41_24_477x321.jpg)

kubespray团队下添加kubespray用户:      

![/images/2018_11_15_11_42_38_492x306.jpg](/images/2018_11_15_11_42_38_492x306.jpg)

创建一个命名空间用于存放kubespray部署镜像:    

![/images/2018_11_15_11_43_40_907x344.jpg](/images/2018_11_15_11_43_40_907x344.jpg)

此刻Dashboard上可以看到我们刚才进行的操作。而在外部则可以通过docker
login来登录到此仓库。    

现在关闭docker-compose启动的容器，备份好关键目录：    

```
# cd /usr/local/compose/
# docker-compose down
Stopping compose_nginx_1_3541e93c08a9      ... done
Stopping compose_background_1_42f1644b8fea ... done
Stopping compose_registry_1_e6eb6ee23d0a   ... done
Stopping compose_portus_1_90c30953f8b0     ... done
Stopping compose_db_1_45dc41479cee         ... done
Removing compose_nginx_1_3541e93c08a9      ... done
Removing compose_background_1_42f1644b8fea ... done
Removing compose_registry_1_e6eb6ee23d0a   ... done
Removing compose_portus_1_90c30953f8b0     ... done
Removing compose_db_1_45dc41479cee         ... done
Removing network compose_default
# cd /var/lib
# tar cJvf portus.tar.xz portus/
# ls -l -h portus.tar.xz 
-rw-r--r-- 1 root root 687K Nov 15 03:49 portus.tar.xz
```
由上面可见，portus下现在没有任何上传的镜像及数据文件，整个目录压缩后仅1M不到的空间。    

我们也需要portus运行所需要的所有镜像，使用下列命令打包成一个压缩后的镜像,
以便我们在编译ISO时使用:    

```
# docker save $(docker images -q) -o portus_combine.tar
# ls -l -h portus_combine.tar 
-rw------- 1 root root 584M Nov 15 03:58 portus_combine.tar
```

由章节1我们得到用于制作Portus纯净仓库的文件，
`portus_combine.tar`和`portus.tar.xz`，用于后续的部署ISO编译使用。    

#### 1.1 vagrant box引出
将上述的文件放到`/home/vagrant`目录， 并更改root的密码为`txxxxxxr`，
打包该虚拟机，以后我们将直接由vagrant box来批量执行.    

```
# pwd
/home/vagrant
# ls
compose.tar.gz  portus_combine.tar  portus.tar.xz
```
打包该vagrant实例为box:    

```
# vagrant status
Current machine states:

default                   poweroff (virtualbox)

The VM is powered off. To restart the VM, simply run `vagrant up`
# vagrant package --output portusBase.box
# ls -l -h portusBase.box
-rw-r--r-- 1 dash root 1.7G Nov 15 12:27 portusBase.box
```

### 2. kubespray容器镜像
VPS改写kubespray脚本，取回需要的容器镜像.   

以下是本地镜像:    
取回后的所有容器镜像，打包到`/vagrant`目录下:    

```
# docker save $(docker images -q) -o kubespray_images.tar
# ls -l -h kubespray_images.tar 
-rw------- 1 root root 4.9G Nov 14 20:37 kubespray_images.tar
```

### 3. Portus部署仓库制作
由纯净版的box启动虚拟机，加载上一章制作出来容器镜像。     

