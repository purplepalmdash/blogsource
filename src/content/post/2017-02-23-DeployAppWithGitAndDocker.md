+++
date = "2017-02-23T11:06:30+08:00"
categories = ["Linux"]
keywords = ["DeployAppWithGitAndDocker"]
description = "Deploy apps using git and docker"
title = "用Git和Docker快速部署应用程序"

+++
### 参考资料
《Docker基础与实战》第八章 (韩) 李在弘。    

### 环境
本机, ArchLinux.   
服务器, DigitalOcean, Ubuntu14.04.    
之所以选择DO的主机，是因为它位于墙外，不会有防火墙的阻挡，每次都能编译成功。    

### 本机配置
#### Git
ArchLinux上安装`sudo pacman -S git`， 之后可以配置global:    

```
$ git config --global user.name xxxx
$ git config --global user.email xxxx@gamil.com
```
#### app.js
用Node.js编写一个简单的Web服务器，返回`Hello Docker`:    

```
var express = require('express');
var app = express();

app.get(['/', '/index.html'], function (req, res) {
  res.send('Hello Docker1');
});

app.listen(80);
```
#### package.json
用于描述运行该程序的依赖关系:    

```
{
  "name": "exampleapp",
  "description": "Hello Docker",
  "version": "0.0.1",
  "dependencies": {
    "express": "4.4.x"
  }
}
```
#### Dockerfile
用于运行该APP的Docker容器定义如下:    

```
FROM ubuntu:14.04

RUN apt-get update
RUN apt-get install -y nodejs npm

ADD app.js /var/www/app.js
ADD package.json /var/www/package.json

WORKDIR /var/www
RUN npm install

CMD nodejs app.js
```
### DO服务器配置
#### Git配置
需要安装Docker， Docker的安装方法不再重复，安装git, 安装方法也不再重复.    

创建服务器端的git仓库:    

```
$ cd ~
$ git init exampleapp
$ cd exampleapp
$ git config receive.denycurrentbranch ignore
```
配置成`receive.denycurrentbranch ignore`是为了便于从开发者PC接收推送的源代码.    
#### Git Hook配置
在`~/exampleapp/.git/hooks`目录下，创建一个`post-receive`文件，内容如下:    

```
#!/bin/bash

APP_NAME=exampleapp
APP_DIR=$HOME/$APP_NAME
REVISION=$(expr substr $(git rev-parse --verify HEAD) 1 7)

GIT_WORK_TREE=$APP_DIR git checkout -f

cd $APP_DIR
docker build --tag $APP_NAME:$REVISION .
docker stop $APP_NAME
docker rm $APP_NAME
docker run -d --name $APP_NAME -p 8068:80 $APP_NAME:$REVISION
```
保存并设置可执行权限:    

```
$ chmod +x ~/exampleapp/.git/hooks/post-receive
```

### 更新流程
在开发机的目录下，添加remote分支:    

```
$ git remote add origin dash@xxx.xxx.xxx.x:exampleapp
```
如果你的DO上主机使用别的端口，例如8894端口，则编辑ssh的配置，为主机指定端口即可:    

```
$ vim ~/.ssh/config
host 1xx.xx.xx.xxx
port 8894
```
此时每次提交的代码都会触发DO上主机的自动化构建过程，且自动侦听8068端口。

### 自动推送到服务器
只需要更改`post-receive`文件即可实现自动推送流程:    

```
#!/bin/bash

APP_NAME=exampleapp
APP_DIR=$HOME/$APP_NAME
REVISION=$(expr substr $(git rev-parse --verify HEAD) 1 7)

+++ REGISTRY=192.168.0.40:5000
+++ APP_SERVERS=(
+++   "pyrasis@192.168.0.101"
+++   "pyrasis@192.168.0.102"
+++ )

GIT_WORK_TREE=$APP_DIR git checkout -f

cd $APP_DIR
docker build --tag $APP_NAME:$REVISION .
docker tag $APP_NAME:$REVISION $REGISTRY/$APP_NAME:$REVISION
docker push $REGISTRY/$APP_NAME:$REVISION

+++ SSH="ssh -o StrictHostKeyChecking=no"
+++ for SERVER in ${APP_SERVERS[@]}
+++ do
+++   $SSH $SERVER docker pull $REGISTRY/$APP_NAME:$REVISION
+++   $SSH $SERVER docker stop $APP_NAME
+++   $SSH $SERVER docker rm $APP_NAME
+++   $SSH $SERVER docker run -d --name $APP_NAME \
+++     -p 80:80 $REGISTRY/$APP_NAME:$REVISION
+++ done
```
上面的代码需要事先实现中转机(即运行git的机器)与目标服务器的ssh免登录设置。
