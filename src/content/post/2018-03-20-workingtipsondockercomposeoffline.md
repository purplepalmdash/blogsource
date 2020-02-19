+++
title = "WorkingTipsOnDockerComposeOfflineInstall"
date = "2018-03-20T09:09:27+08:00"
description = "WorkingTipsOnDockerComposeOfflineInstall"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 步骤
获取所需安装包的步骤如下：    

```
$ sudo docker run -it ubuntu:16.04 /bin/bash
# 进入容器后的操作
# rm -f /etc/apt/apt.conf.d/docker-clean
# apt-get update
# apt-get install docker-compose dpkg-dev
```
获取所有的deb包并拷贝到某一目录下:    

```
# cd /var/cache
# find . | grep deb$ | xargs -I % cp % /root/pkgs
# cd /root/pkgs
# dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
```
将整个目录拷贝到主机目录:    

```
# sudo docker cp d1023d1b0a1f:/root/pkgs /xxxxxx/xxxxx
# tar czvf /xxxx/xxxx.tar.gz /xxxxxx/xxxx
```
### 使用步骤
下载地址: 

```
http://192.192.189.127/big.tar.gz
```

下载所需要的大包，解压以后，得到以下四个文件:    

```
# tar xzvf big.tar.gz
# ls big
images/ pkgs.tar.gz  docker-regsitry.tar.gz devdockerCA.crt
```

取得pkgs.tar.gz, 将其放置在自己的网页服务器目录，如:    
![/images/2018_03_20_09_22_07_724x495.jpg](/images/2018_03_20_09_22_07_724x495.jpg)

配置节点的apt:    

```
# vim /etc/apt/sources.list
deb http://192.192.189.127/pkgs/	/
# apt-update
# apt-get install docker-compose
```
这将安装docker-compose, docker, 安装完毕以后，docker-compose可以正常使用。    

### 载入镜像
载入registry所需的registry:2镜像和nginx镜像:    

```
# docker load<images/1.tar
# docker load<images/2.tar
```
### 解压docker-compose目录，添加服务
解压到指定目录(可为任意目录，这里以/docker-registry为例):    

```
# tar xzvf docker-registry.tar.gz -C /
# ls /docker-regsitry
data	docker-compose.yml nginx
```
创建systemd所需服务:    

```
# vim /etc/systemd/system/docker-compose.service
[Unit]
Description=DockerCompose
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/bin/docker-compose -f /docker-registry/docker-compose.yml up -d

[Install]
WantedBy=multi-user.target
```

启动、使能docker-compose服务:    

```
# sudo systemctl enable docker-compose.service
# sudo systemctl start docker-compose.service
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                          NAMES
50d2ec00e967        nginx:1.9           "nginx -g 'daemon ..."   26 hours ago        Up 3 seconds        80/tcp, 0.0.0.0:443->443/tcp   dockerregistry_nginx_1
e4e2cee1bf21        registry:2          "/entrypoint.sh /e..."   26 hours ago        Up 5 seconds        127.0.0.1:5000->5000/tcp       dockerregistry_registry_1
```
如此即设置好了整个regitry服务。可以通过https://YourIP来访问，签名文件也可以在目录中找到(devdockerCA)。

### 使用镜像仓库
Redhat rh74:    

```
# mkdir -p /etc/docker/certs.d/mirror.xxxx.com
# wget xxxxxx.xxxx.xxx.com/ca.crt
# docker login -u xxxx -p xxxx mirror.xxxx.com
```

