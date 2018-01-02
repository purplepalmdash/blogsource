+++
categories = ["Virtualization"]
date = "2016-12-23T19:01:18+08:00"
description = "Load banlancing in coreos with confd/nginx"
keywords = ["Virtualization"]
title = "LoadBalancingInCoreOS"

+++
这几天一直在把玩CoreOS，
主要参考的是DigitalOcean上的tutorial以及《CoreOS实践之路》这本书，
奈何文章离如今年代已经久远，一直搭建不成功. 几经周折后终于在一篇guideline
的指导下把负载均衡的服务跑通，这里是搭建该服务的步骤和心得。    

参考网址:    

[http://blog.dixo.net/2015/02/load-balancing-with-coreos/](http://blog.dixo.net/2015/02/load-balancing-with-coreos/)    

### 架构图
该网址中列举的的架构图如下:    

![/images/2016_12_23_19_10_52_1450x1029.jpg](/images/2016_12_23_19_10_52_1450x1029.jpg)    

刚开始看到这个图是有点发蒙的，这里简单说一下操作步骤，与图中一一对应.   

1. `CoreOS Machine B`和`Core Machine C`是两个CoreOS系统节点，位于其上分别
运行了两个apache容器，B上的容器监听8001端口，C上的容器监听8002端口。    
2. 两个apache容器将自身的IP和端口发布到etcd服务(etcd.service).    
3. `CoreOS Machine
   A`上运行了三个单元，分别是nginx服务、confdata服务、confd服务.
其中nginx服务在配置好的负载均衡后端上分发http请求。confdata服务主要用于
为nginx配置文件共享数据卷。confd服务查看etcd中元数据的变化，根据这些变化
在共享数据卷中写入新的配置文件.    
4. 详细说明一下confd的作用，A. 发现coreos集群中可用的apache容器. B. 实时生成
nginx.conf文件，并将此文件写入到共享存储. C. 写入完成后，通知docker给nginx发送
一个HUP信号. D. Docker发送HUP信号给nginx容器后，容器将重新加载其配置文件。    

以上就是我对架构图的解读。接下来将一步步来实现这个负载均衡。    

### 先决条件
一个3节点的CoreOS集群    
有效的/etc/environment文件(有时候需要手动生成) 
相关的容器(制作流程见后)    

### 容器镜像制作
#### Apache容器
在某台安装好Docker的机器上，或者直接在CoreOS节点机上，运行:    

```
$ docker run -i -t ubuntu:14.04 /bin/bash
# apt-get update
# apt-get install apache2
# sudo bash
# echo "<h1>Running from Docker on CoreOS</h1>" > /var/www/html/index.html
# exit
```
现在开始打包容器:    

```
$ docker ps -l
$ docker commit  container_ID dash/apache
$ docker save dash/apache>myapache.tar
```
将生成的tar文件分发到所有CoreOS节点上，使用`docker load<./myapache.tar`加载之.    

#### nginx容器
直接使用官方的nginx:latest即可    

#### nginx-lb容器
使用Dockerfile生成:    

```
$ vim Dockerfile
FROM ubuntu:14.04

RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get -y install curl && \
    curl -o /usr/bin/confd -L https://github.com/kelseyhightower/confd/releases/download/v0.7.1/confd-0.7.1-linux-amd64 && \
    chmod 755 /usr/bin/confd  && curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -

ADD etc/confd/ /etc/confd

CMD /usr/bin/confd -interval=60 -node=http://$COREOS_PRIVATE_IPV4:4001
```
编译方法为`sudo docker build -t myconfd .`     

而后的打包和解包方法同上。奇怪的是，我制作的镜像到最后居然无法启动，所以直接pull
了作者制作的镜像:    

```
$ sudo docker pull lordelph/confd-demo
```
### 创建apache服务
创建apache.service配置文件如下:    

```
$ vim apache.service 
[Unit]
Description=Basic web service port %i
After=docker.service
Requires=docker.service

[Service]
EnvironmentFile=/etc/environment
ExecStartPre=-/usr/bin/docker kill apache-%i
ExecStartPre=-/usr/bin/docker rm apache-%i
ExecStartPre=/usr/bin/etcdctl set /test/apache-%i ${COREOS_PRIVATE_IPV4}:%i
ExecStart=/usr/bin/docker run --rm --name apache-%i -p ${COREOS_PRIVATE_IPV4}:%i:80 dash/apache  /usr/sbin/apache2ctl -D FOREGROUND
ExecStop=/usr/bin/etcdctl rm /test/apache-%i
ExecStop=/usr/bin/docker stop -t 3 apache-%i
```
这里要说明的几点是: 在创建该服务前将清空该节点上所有的同名容器运行历史，并在etcd中设置
一个键值为`/test/apache-%i`的数据条目。关闭该服务则停止容器的运行，并删除该键值.    

`COREOS_PRIVATE_IPV4`这个值由`/etc/environment`中给出，因而事先需要
在每个节点上核查是否存在该值:    

```
core@coreos1 ~/lb $ cat /etc/environment 
COREOS_PUBLIC_IPV4=172.17.8.201
COREOS_PRIVATE_IPV4=172.17.8.201
```

`apache-%i`里的端口参数可以在启动时给定, %i占位符将被@后的值所代替。创建服务步骤如下:    

```
$ ln -s apache.service apache@8001.service 
$ ln -s apache.service apache@8002.service 
$ fleetctl start apache@8001.service
$ fleetctl start apache@8002.service
```
查看运行的服务单元:    

```
$ fleetctl list-units
UNIT			MACHINE				ACTIVE	SUB
apache@8001.service	bea5741d.../172.17.8.203	active	running
apache@8002.service	dd464e69.../172.17.8.202	active	running
```
检查etcd中新添加的值:    

```
$ etcdctl ls /test
/test/apache-8001
/test/apache-8002
```
key对应的value:    

```
$ etcdctl get /test/apache-8001
172.17.8.203:8001
```
### confd数据卷
用于创建confd数据卷的service定义文件如下:     

```
$ vim confdata.service 
[Unit]
Description=Configuration Data Volume Service
After=docker.service
Requires=docker.service

[Service]
EnvironmentFile=/etc/environment

#we aren't a normal service, we just need to ensure that a data volume
#exists, and create one if it doesn't
Type=oneshot
RemainAfterExit=yes

ExecStartPre=-/usr/bin/docker rm conf-data
ExecStart=/usr/bin/docker run -v /etc/nginx --name conf-data nginx echo "created new data container"
```
这里的技巧在于`oneshot`属性，它告知systemd我们只希望该服务运行且只运行一次。`RemainAfterExit`告诉
服务正常退出，因而我们可以在正常推出的服务上使用它创建的数据卷。    

运行该服务:    

```
$ fleetctl start confdata.service
$ fleetctl list-units
UNIT			MACHINE				ACTIVE	SUB
apache@8001.service	bea5741d.../172.17.8.203	active	running
apache@8002.service	dd464e69.../172.17.8.202	active	running
confdata.service	f22aee5d.../172.17.8.201	active	exited
```

可以使用以下一系列命令检查我们生成的数据卷, 注意要使用root才能看到该卷里的实际内容:     

```
$ docker volume ls
DRIVER              VOLUME NAME
local               c19a35d3db97340272f5d191b166710f6fbb1d717225d9762417fa51c7a56b1f
core@coreos1 ~/lb $ docker volume inspect c19a35d3db97340272f5d191b166710f6fbb1d717225d9762417fa51c7a56b1f
[
    {
        "Name": "c19a35d3db97340272f5d191b166710f6fbb1d717225d9762417fa51c7a56b1f",
        "Driver": "local",
        "Mountpoint": "/var/lib/docker/volumes/c19a35d3db97340272f5d191b166710f6fbb1d717225d9762417fa51c7a56b1f/_data",
        "Labels": null
    }
]
core@coreos1 ~/lb $ sudo bash
coreos1 lb # cd /var/lib/docker/volumes/c19a35d3db97340272f5d191b166710f6fbb1d717225d9762417fa51c7a56b1f/_data
coreos1 _data # ls
conf.d  fastcgi_params  koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params  uwsgi_params  win-utf
```

### confd服务
confd.service文件定义如下:    

```
$ vim confd.service 
[Unit]
Description=Configuration Service

#our data volume must be ready
After=confdata.service
Requires=confdata.service


[Service]
EnvironmentFile=/etc/environment


#kill any existing confd
ExecStartPre=-/usr/bin/docker kill %n
ExecStartPre=-/usr/bin/docker rm %n

#preload container...this ensures we fail if our registry is down and we can't
#obtain the build we're expecting

#we need to provide our confd container with the IP it can reach etcd
#on, the docker socket so it send HUP signals to nginx, and our data volume
ExecStart=/usr/bin/docker run --rm \
  -e COREOS_PRIVATE_IPV4=${COREOS_PRIVATE_IPV4} \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --volumes-from=conf-data \
  --name %n \
  lordelph/confd-demo

ExecStop=/usr/bin/docker stop -t 3 %n
Restart=on-failure

[X-Fleet]
#we need to be on the same machine as confdata.service
MachineOf=confdata.service
```
注意，我们这里直接使用了网页中作者制作的镜像，指定了`conf-data`数据卷(`--volumes-from=conf-data`).    

```
$ fleetctl list-units
UNIT			MACHINE				ACTIVE	SUB
apache@8001.service	bea5741d.../172.17.8.203	active	running
apache@8002.service	dd464e69.../172.17.8.202	active	running
confd.service		f22aee5d.../172.17.8.201	active	running
confdata.service	f22aee5d.../172.17.8.201	active	exited
```
检查配置文件是否被更新:    

```
coreos1 _data # grep -A6 'upstream backend' ./nginx.conf 
    upstream backend {
        
            server 172.17.8.202:8002;
        
            server 172.17.8.203:8001;
        
    }
coreos1 _data # pwd
/var/lib/docker/volumes/c19a35d3db97340272f5d191b166710f6fbb1d717225d9762417fa51c7a56b1f/_data
```
或者直接进入容器检查:    

```
# docker run --rm -ti --volumes-from=conf-data nginx \
# grep -A6 'upstream backend' /etc/nginx/nginx.conf
    upstream backend {
        
            server 172.17.8.101:8001;
        
            server 172.17.8.102:8002;
        
    }
```
### nginx服务
这个服务很简单，定义文件如下:     

```
$ vim nginx.service 
[Unit]
Description=Nginx Service
After=confd.service

#we won't want it to require the service - that would stop us restarting
#it, which is safe
#Requires=confd.service

[Service]
EnvironmentFile=/etc/environment
ExecStartPre=-/usr/bin/docker kill %n
ExecStartPre=-/usr/bin/docker rm %n
#ExecStartPre=/usr/bin/docker pull nginx
ExecStart=/usr/bin/docker run --name %n -p 80:80 --volumes-from=conf-data nginx
ExecStop=/usr/bin/docker stop -t 3 %n
Restart=on-failure

[X-Fleet]
#we need to be on the same machine as confdata
MachineOf=confdata.service
```
这个服务被定义为只能在运行了confdata.service的机器上运行，实际上因为我们为了使用
`conf-data`数据卷。     
启动并检查服务:     

```
$ fleetctl start nginx.service
$ fleetctl list-units
UNIT			MACHINE				ACTIVE	SUB
apache@8001.service	bea5741d.../172.17.8.203	active	running
apache@8002.service	dd464e69.../172.17.8.202	active	running
confd.service		f22aee5d.../172.17.8.201	active	running
confdata.service	f22aee5d.../172.17.8.201	active	exited
nginx.service		f22aee5d.../172.17.8.201	active	running
```

### 测试
打开浏览器，访问`172.17.8.201`，即可看到apache服务的页面。    
可以使用tcpdump来查看流量走向，然而需要安装toolbox: `/usr/bin/toolbox`.   
