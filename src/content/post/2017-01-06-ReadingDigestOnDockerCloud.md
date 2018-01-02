+++
date = "2017-01-06T15:42:25+08:00"
categories = ["Virtualization"]
keywords = ["Virtualization"]
description = "Reading Digests on Docker Cloud"
title = "DockerCloudReadingDigests"

+++
### 第一章
Docker镜像准备:    

```
$ sudo docker pull redis
$ sudo docker pull django
$ sudo docker pull haproxy
$ sudo docker pull ubuntu
```
应用栈节点架构:    

启动redis-master容器节点, 两个redis-slave容器节点启动时连接到redis-master上面,
两个app容器节点启动时连接到redis-master上面, haproxy容器结点启动时连接到两个app结点上面。    

容器的启动顺序为：redis-master -> redis-slave -> APP -> HAProxy.    

Redis Master:    

```
$ sudo docker run -it --name redis-master redis /bin/bash
root@4e4e597ffcb6:/data# 
```
Redis Slave1/Slave2:    

```
$ sudo docker run -it --name redis-slave1 --link redis-master:master redis /bin/bash
$ sudo docker run -it --name redis-slave2 --link redis-master:master redis /bin/bash
```
App1, App2:    

```
$ mkdir -p ~/Projects/Django/App1
$ mkdir -p ~/Projects/Django/App2
$ sudo docker run -it --name APP1 --link redis-master:db -v ~/Projects/Django/App1:/user/src/app django /bin/bash
$ sudo docker run -it --name APP2 --link redis-master:db -v ~/Projects/Django/App2:/user/src/app django /bin/bash
```
HAProxy:    

```
$ mkdir -p ~/Projects/HAProxy
$ sudo docker run -it --name HAProxy --link APP1:APP1 --link APP2:APP2 -p 6301:6301 -v ~/Projects/HAProxy:/tmp haproxy /bin/bash
```
检查source目录挂载:    

```
$ sudo docker ps | grep master
4e4e597ffcb6        redis                            "docker-entrypoint.sh"
43 minutes ago      Up 43 minutes       6379/tcp                 redis-master
$ sudo docker inspect 4e4e | grep Source
"Source":
"/var1/DockerRepo/docker/volumes/1dc31736abd411569cfbc51c6624125867883b44733f40113e2f918770843438/_data",
```
下载redis.conf文件:    

```
$ wget http://download.redis.io/redis-stable/redis.conf
$ cp redis.conf /var1/DockerRepo/docker/volumes/1dc31736abd411569cfbc51c6624125867883b44733f40113e2f918770843438/_data
```
修改redis.conf内容:    

```
bind 0.0.0.0
```
进入到容器后的操作:    

```
# mkdir /var/lib/redis
# cd /data
# cp redis.conf /usr/local/bin
# cd /usr/local/bin
# redis-server redis.conf
```
Slave节点的配置:    

```
# docker inspect f88a69ca2b1e | grep Source
                "Source": "/var1/DockerRepo/docker/volumes/6b211a6e469f4864a723dd7531f49674eeb9af9509ddcb75de8f18f4a677b85f/_data",
# cd /var1/DockerRepo/docker/volumes/6b211a6e469f4864a723dd7531f49674eeb9af9509ddcb75de8f18f4a677b85f/_data
# cp /home/dash/redis.conf  ./
# vim redis.conf 
slaveof master 6379
# docker inspect 14bbec8a900b | grep Source
                "Source": "/var1/DockerRepo/docker/volumes/a51e5a23a9b2e36350325aac6c533a48beb34d566b08773d48a2a3273c786d42/_data",
# cp redis.conf /var1/DockerRepo/docker/volumes/a51e5a23a9b2e36350325aac6c533a48beb34d566b08773d48a2a3273c786d42/_data
```
启动流程和Master一样。    

Django配置, 注意要在我们映射的目录下:    

```
root@7243f0d5a231:/user/src/app# mkdir dockerweb
root@7243f0d5a231:/user/src/app# cd dockerweb/
root@7243f0d5a231:/user/src/app/dockerweb# cd redisweb/
root@7243f0d5a231:/user/src/app/dockerweb/redisweb# python manage.py startapp helloworld
```
修改django文件:    

```
$ pwd
/user/src/app
$ vim dockerweb/redisweb/helloworld/views.py 
from django.shortcuts import render
from django.http import HttpResponse

#Create your views here

import redis

def hello(request):
    str = redis.__file__
    str += "<br>"
    r = redis.Redis(host='db', port=6379, db=0)
    info = r.info()
    str += ("Set Hi <br>")
    r.set('Hi', 'HelloWorld-APP1')
    str += ("Get Hi: %s <br>" % r.get('Hi'))
    str += ("Redis Info: <br>")
    str += ("Key: Info Value")
    for key in info:
        str += ("%s: %s <br>" % (key, info[key]))
    return HttpResponse(str)
```
添加helloworld程序到`INSTALLED_APPS`选项下:    

```
$ vim dockerweb/redisweb/redisweb/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'helloworld',
]
```
修改urls.py:    

```
$ vim dockerweb/redisweb/redisweb/urls.py 
from django.conf.urls import url
from django.contrib import admin
from helloworld.views import hello

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^helloworld$', hello),
]
```
App1和App2的差别在于views.py里改掉这一行:    

```
    - r.set('Hi', 'HelloWorld-APP1')
    + r.set('Hi', 'HelloWorld-APP2')
```
之后运行:    

```
# python manage.py makemigrations
# python manage.py migrate
# python manage.py createsuperuser
```
分别运行:    

```
#### APP1
# python manage.py runserver 0.0.0.0:8001
#### APP2
# python manage.py runserver 0.0.0.0:8002
```
HAProxy配置:    

```
# cd ~/Projects/HAProxy
# vim haproxy.cfg
global 
  log 127.0.0.1 local0
  maxconn 4096
  chroot /usr/local/sbin
  daemon
  nbproc 4
  pidfile /usr/local/sbin/haproxy.pid

defaults
  log 127.0.0.1	local3
  mode http 
  option dontlognull 
  option redispatch
  retries 2
  maxconn 2000
  balance roundrobin
  timeout connect 5000ms
  timeout client 50000ms
  timeout server 50000ms

listen redis_proxy
  bind 0.0.0.0:6301
  stats enable
  stats uri /haproxy-stats
      server APP1 APP1:8001 check inter 2000 rise 2 fall 5 
      server APP2 APP2:8002 check inter 2000 rise 2 fall 5 
```
在haproxy容器里:   

```
# cd /tmp
# cp haproxy.cfg /usr/local/sbin/
# cd /usr/local/sbin
# haproxy -f haproxy.cfg
```
现在验证:    

`http://127.0.0.1:6301/helloworld`    

![/images/2017_01_06_18_45_40_627x330.jpg](/images/2017_01_06_18_45_40_627x330.jpg)    

`http://127.0.0.1:6301/haproxy-stats`:    

![/images/2017_01_06_18_46_45_634x397.jpg](/images/2017_01_06_18_46_45_634x397.jpg)    


