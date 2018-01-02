+++
title = "将网站转化为Android应用程序"
date = "2017-01-13T19:22:37+08:00"
categories = ["Embedded"]
keywords = ["Embedded"]
description = "Convert Website to android app"

+++
写了几年博客，博文已有数百篇，于是我想能不能把每次需要上网搜索的过程改为在手机端本地查找呢？    

看了下网上的解决方案，参考了以下这个，可以作为静态网站APP化的第一步。    

参考链接:    
[http://www.vetbossel.in/convert-website-to-android-application/](http://www.vetbossel.in/convert-website-to-android-application/)    

### 构建开发环境
ArchLinux下安装android-studio的方法是通过yaourt:    

```
$ yaourt -S android-studio
```
需要注意的是，安装过程中需要访问Google网站，因而需要代理（你懂的）。而安装完毕后，需要安装本地模拟器，下载
img也需要用到代理。确保你翻墙的速度够快。    

运行模拟器会出错，可以通过以下两个步骤来纠正:    

```
$ yaourt glxinfo
$ cd ~/Android/Sdk/tools/lib64/libstdc++/
$ mv libstdc++.so.6 libstdc++.so.6.back
$ ln -s /usr/lib/libstdc++.so.6 ~/Android/Sdk/tools/lib64/libstdc++/libstdc++.so.6
```
### 开发
开发步骤可以参考原文。   

值得注意的是，在`MainActivity.java`文件中，第一行需要改正为正确的包名，否则会出现编译错误。    

```
- package com.example.vetri.websitetoapplication;
+ package com.example.dash.websitetoapplication;
```
### 执行
实际运行的效果如下:    

![/images/2017_01_13_19_31_05_373x680.jpg](/images/2017_01_13_19_31_05_373x680.jpg)    

### 想法
在Android端已经可以有简单的Web服务器，考虑把hugo生成的网站内容上传到Android手机内的Web服务器上。    

因为网站是本地化的，所以可以去掉一切不需要的插件，例如disqus之类。
