+++
categories = ["Linux"]
date = "2016-12-11T11:23:06+08:00"
description = "Install Java Offline On Ubuntu14.04"
keywords = ["Linux"]
title = "OfflineInstallJavaOnUbuntu1404"

+++
在ubuntu14.04上搭建mesos时，需要安装jdk8或jdk9，然而官方仓库中没有类似的选项，
因而有两种work-around，第一种是手动安装jdk， 第二种是使用第三方库安装.    

### 手动安装
下载安装包:    

```
$ wget --header "Cookie: oraclelicense=accept-securebackup-cookie"
http://download.oracle.com/otn-pub/java/jdk/8u111-b14/jdk-8u111-linux-x64.tar.gz
```
如果是jdk9，则下载安装包为:    

```
$ wget
http://www.java.net/download/java/jdk9/archive/140/binaries/jdk-9-ea+140_linux-x64_bin.tar.gz
```
下载完毕后，安装步骤如下:    

```
$ sudo bash
# tar -zxf jdk-9-ea+140_linux-x64_bin.tar.gz -C /opt/jdk/
# update-alternatives --install /usr/bin/java java /opt/jdk/jdk-9/bin/java 100
# update-alternatives --install /usr/bin/javac javac /opt/jdk/jdk-9/bin/javac 100
# update-alternatives --display java
# update-alternatives --display javac
```
### 第三方库
激活ppa库如下:    

```
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
$ sudo apt-get install oracle-java8-set-default
```
在安装过程中需要从oracle官方网站下载安装包，为了避免重复下载过程，可以将安装包直接拷贝到
`/var/cache/oracle-jdk9-installer`目录或者`/var/cache/oracle-jdk8-installer`目录。    


