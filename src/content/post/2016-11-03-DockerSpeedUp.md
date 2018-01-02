+++
categories = ["Linux"]
date = "2016-11-03T17:23:23+08:00"
description = "Speed up docker deployment"
keywords = ["Linux"]
title = "DockerSpeedUp"

+++
### Docker Registry
因为众所周知的原因，在国内下载Docker镜像会很慢，所以我们更改docker的配置，让它使用
daocloud的加速服务:    

ArchLinux下，编辑以下文件，或者你可以通过`sudo systemctl edit docker.service`来配置以下
文件:    

```
# vim /etc/systemd/system/docker.service.d/override.conf 
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --registry-mirror=http://1a653205.m.daocloud.io -H fd://
```
现在重新加载服务并重新启动`docker.service`：    

```
# systemctl daemon-reload
# systemctl restart docker.service
```
从此以后，每一次docker pull都会使用到daocloud提供的加速服务。    

### 本地私有registry
首先pull下来如下镜像(这两个其实是一个镜像):    

```
# docker pull registry:latest
# docker pull registry:2
```
运行registry实例(注意更改volumn映射):    

```
# docker run -d -p 5000:5000 --restart=always --name registry -v
/root/DockerRegistry:/var/lib/registry registry:2
```
检查实例运行情况:    

```
➜  ~ sudo docker ps
CONTAINER ID        IMAGE                            COMMAND                  CREATED
STATUS              PORTS                    NAMES
b16b27933709        registry:2                       "/entrypoint.sh /etc/"   7 hours
ago         Up 2 hours          0.0.0.0:5000->5000/tcp   registry
```
为了让这个服务每次都启动，我们来编写一个systemd服务:    

```
# vim /usr/lib/systemd/system/DockerRegistry.service 
[Unit]
Description=DockerRegistry container
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker start -a registry
ExecStop=/usr/bin/docker stop -t 2 registry

[Install]
WantedBy=multi-user.target
```
现在使能该服务:    

```
# systemctl enable DockerRegistry.service
Created symlink /etc/systemd/system/multi-user.target.wants/DockerRegistry.service →
/usr/lib/systemd/system/DockerRegistry.service.
```
### 本地私有Registry使用方法
将获取到的image tag到私有registry(tag):   

```
# docker tag ubuntu:16.04 localhost:5000/ubuntu:16.04
```
将获取到的image push到私有registry(push):    

```
# docker push localhost:5000/ubuntu:16.04
```

### 在agent机器上启用本地registry仓库
在systemd类型的机器上，例如Ubuntu16.04上，使用以下命令，添加`192.168.177.11:5000`这个私有仓库：

```
# sed -i 's#fd://#fd:// --registry-mirror http://192.168.177.11:5000
--insecure-registry 192.168.177.11:5000#' /lib/systemd/system/docker.service
# systemctl daemon-reload
# systemctl restart docker
```

添加完毕后，检查docker运行参数:    

```
# ps -ef | grep docker
root      4253     1  2 21:48 ?        00:00:00 /usr/bin/dockerd -H fd://
--registry-mirror http://192.168.177.11:5000 --insecure-registry
192.168.177.11:5000
root      4260  4253  0 21:48 ?        00:00:00 docker-containerd -l
unix:///var/run/docker/libcontainerd/docker-containerd.sock --shim
docker-containerd-shim --metrics-interval=0 --start-timeout 2m --state-dir
/var/run/docker/libcontainerd/containerd --runtime docker-runc
```
pull一个已经被放入仓库里的镜像，速度非常快:    

```
# docker pull 192.168.177.11:5000/fedora:latest
latest: Pulling from fedora
41b563eedcb0: Pull complete 
Digest:
sha256:cc4323bf2ee99b414600605e42d1888c4c1b0a08f5793a379c1238af4a44315d
Status: Downloaded newer image for 192.168.177.11:5000/fedora:latest
```
检查安装的镜像:     

```
$ sudo docker run -it 192.168.177.11:5000/fedora /bin/bash
[root@810f44567c87 /]# cat /etc/redhat-release 
Fedora release 24 (Twenty Four)
```
