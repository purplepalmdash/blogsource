+++
title = "CustomizeDockerISO"
date = "2018-07-20T09:07:43+08:00"
description = "CustomizeDockerISO"
keywords = ["Linux"]
categories = ["Technology"]
+++
单位里一些同事需要一个开箱即用的Docker环境，以下是制作自启动Docker的ISO制作过程。
### 准备
准备一台新安装的Ubuntu16.04机器，在其中安装docker, docker
load需要定制的镜像。而后，保存/var/lib/docker/目录下的条目，简而言之，就是将`/var/lib/docker`压包。   
### 定制化ISO
从基础镜像起步，之前我已经定制了1604_pure.iso，
里面已经安装docker/docker-compose，并内置了用于运行portus（一个容器镜像仓库）所需的镜像文件，现在只需要从其中替换掉镜像文件即可。   


