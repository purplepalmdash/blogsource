+++
categories = ["Tools"]
date = "2016-07-08T21:35:19+08:00"
description = "On building dockerized piwigo website"
keywords = []
title = "搭建基于Docker的Piwigo照片管理网站"

+++
最近和小伙伴一起出去玩的次数比较多，免不了要拍下不少照片。考虑到照片共享的便捷性，特地调研了
几个关于照片共享的方法，最后打算基于Piwigo来搭建，以下是详细的步骤。    

### 从Iphone同步图片到ArchLinux
参考了[https://wiki.archlinux.org/index.php/IPod](https://wiki.archlinux.org/index.php/IPod)    

```
$ sudo pacman -S ifuse usbmuxd libplist
$ lsmod | grep -i fuse
fuse                   94208  3
$ ifuse ~/iphone 
$ cd ~/iphone 
$ ls
Books  DCIM  Downloads  MediaAnalysis  PhotoData  PhotoStreamsData
Photos  Purchases  Radio  Recordings  iTunes_Control
```
`DCIM/100APPLE`下即为我们Iphone里所储存的图片。可以直接拷贝到本地。  

### Piwigo
有了容器以后，很多配置的工作就完全被简化了。以下是步骤：    

```
$ sudo docker pull mathieuruellan/piwigo
$ sudo docker pull mysql:5.5
```     
而后，在某个目录下，创建一个fig.yml文件, fig可以通过`pip install fig`来安装.    

```
$ pwd
/home/dash/Code/piwigo
$ cat fig.yml
mysqlpiwigo:
   image: mysql:5.5 
   volumes:
      - /home/dash/piwigo/mysql/:/var/lib/mysql 
   environment:
      - MYSQL_ROOT_PASSWORD=XXXXXXXX
      - MYSQL_DATABASE=piwigo
      - MYSQL_USER=piwigo
      - MYSQL_PASSWORD=XXXXXX
piwigo:
   image: mathieuruellan/piwigo
   links:
      - mysqlpiwigo:mysql 
   volumes:
      - /home/dash/piwigo/data/galleries:/var/www/galleries
      - /home/dash/piwigo/data/local:/var/www/local
      - /home/dash/piwigo/data/plugins:/var/www/plugins
      - /home/dash/piwigo/data/themes:/var/www/themes
      - /home/dash/piwigo/cache:/var/www/_data/i
      - /home/dash/piwigo/upload:/var/www/upload"
      - /var/log
      - /var/log/piwigo:/var/log/apache2
   ports:
      - "8964:80"
   hostname: piwigo
   domainname: localhost
```
写好以上的配置文件以后，在该目录下运行`sudo fig up -d`即可将Piwigo运行起来。     
运行的结果检查：    

```
$ sudo docker ps
CONTAINER ID        IMAGE                   COMMAND                  CREATED             STATUS              PORTS                  NAMES
9575cd9dbbae        mathieuruellan/piwigo   "/bin/sh -c /entrypoi"   40 minutes ago      Up 40 minutes       0.0.0.0:8964->80/tcp   piwigo_piwigo_1
31f7ecc60985        mysql:5.5               "docker-entrypoint.sh"   41 minutes ago      Up 41 minutes       3306/tcp               piwigo_mysqlpiwigo_1
```
容器启动完毕后，就可以开始配置网站了。   

###配置piwigo

打开浏览器，访问`http://localhost:8964`， 即可访问到piwigo的初始配置页面，如下图:    

![/images/2016_07_08_21_04_21_822x818.jpg](/images/2016_07_08_21_04_21_822x818.jpg)    

这里要注意的是，MySQL主机名字应该填我们在fig配置文件中定义出的`mysql`。数据库名称填piwigo，
这些同样在启动piwigo容器的fig文件中被定义。    

配置好以后，进入到后台，可以选择上传图片，如图:    

![/images/2016_07_08_22_13_40_1318x660.jpg](/images/2016_07_08_22_13_40_1318x660.jpg)    

400多M的图片大概需要用1分钟时间上传完毕。    

好了，一个完全可控的图像分享站点OK了，你可以用它来管理你的家庭照片。一切就只需要两个容器而已。

