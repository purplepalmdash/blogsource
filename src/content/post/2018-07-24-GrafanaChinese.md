+++
title = "GrafanaChinese"
date = "2018-07-24T12:26:41+08:00"
description = "GrafanaChinese"
keywords = ["Linux"]
categories = ["Technology"]
+++
公司内部要用到Grafana肯定会有各种阻力，最大阻力恐怕来自于很多同事英文不好。因而参考了网上的一个教程做grafana的汉化。实际的汉化我也没有做完，无非是做一个框架在这里，以后如果研发压力大，就直接铺人在上头做做翻译。    

环境部署:    

```
$ git clone git://github.com/grafana/grafana
$ git checkout tags/v5.2.1 -b grafana_local
```
安装cnpm开发环境:    

```
$ sudo npm cache clean --force
$ sudo npm install -g cnpm --registry=https://registry.npm.taobao.org
$ which cnpm
/usr/bin/cnpm
```
开发环境初始化:    

```
$ cnpm install
$ npm start
```
此时，生成的代码将运行于
`localhost:3333`端口，我们需要一个本地运行的3000端口的grafana服务器来查看更改。   

需要拷贝出已经运行容器里的`/usr/share/grafana`下的内容:    

```
$ sudo docker cp 66daaf6770b9:/usr/share/grafana .
```
而后更改`conf/defaults.ini`下的条目:    

```
app_mode = development
static_root_path = /public
```
现在重新启动该容器，注意需要挂载新的盘符至其中:    

```
sudo docker run -d -p 3000:3000 -e "GF_SECURITY_ADMIN_PASSWORD=admin_password" -v /media/sda5/grafana_db:/var/lib/grafana -v /home/xxxx/grafana/grafana:/usr/share/grafana -v /home/xxxx/Code/grafana/grafana/public:/public grafana/grafana
```
此时对代码的任一修改都可以很快体现在最终的`http://localhost:3000`网站中。    

修改的一个截图如下:    

![/images/2018_07_24_14_32_30_586x522.jpg](/images/2018_07_24_14_32_30_586x522.jpg)
