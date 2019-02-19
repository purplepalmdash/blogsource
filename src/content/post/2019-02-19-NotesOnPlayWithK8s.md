+++
title = "NotesOnPlayWithK8s"
date = "2019-02-19T14:59:15+08:00"
description = "NotesOnPlayWithK8s"
keywords = ["Linux"]
categories = ["Linux"]
+++
早先在2018年做了一个离线版本的playwithk8s, 主要参考了:    

https://labs.play-with-k8s.com/   

以及    

https://training.play-with-kubernetes.com/kubernetes-workshop/    

当时还写了一系列教程并形成了一个离线部署的ISO。那个ISO前几天有同事用，反映跑不起来。看了下问题，总结如下.    

### dns问题
安装完的系统中，dnsmasq不能使用，需要通过以下步骤来修正:    

```
# vim /etc/systemd/resolved.conf
DNSStubListener=no
# systemctl disable systemd-resolved.service
# systemctl stop systemd-resolved.service
# echo nameserver 192.168.0.15>/etc/resolv.conf
# apt-get install -y dnsmasq
# systemctl enable dnsmasq
# vim /etc/dnsmasq.conf
address=/192.168.122.151/192.168.122.151
address=/localhost/127.0.0.1
# chattr +i /etc/resolv.conf
# chattr -e /etc/resolv.conf
# ufw disable
# docker swarm leave
# docker swarm init
```
这样下来可以访问到界面，    

![/images/2019_02_19_15_05_45_1065x630.jpg](/images/2019_02_19_15_05_45_1065x630.jpg)

### kubeadm卡顿
卡在init的时候，现象见上图。   

问题分析，在docker版本为17.12.1-ce的系统上可正常运行。     

ISO安装出来的docker版本为18.06.0-ce.    

是否是docker版本与kubeadm不兼容?    

kubeadm生成的文件(/etc/kubernetes):     

![/images/2019_02_19_15_08_49_565x229.jpg](/images/2019_02_19_15_08_49_565x229.jpg)


对比kube-apiserver.yaml, 发现imagePullPolicy的不同:    

![/images/2019_02_19_15_10_09_1106x395.jpg](/images/2019_02_19_15_10_09_1106x395.jpg)

这点也是很让人奇怪的，为什么同样的容器镜像版本,
franela/k8s:latest会有如此的不同呢？
一样的容器镜像派生出来的实例，应该说其内置的kubeadm生成的yaml编排文件是一样的，但是在我们的系统上，更新版本的docker下生成的yaml文件未带IfNotPresent的选项，导致kubeadm认为镜像不存在，报了超时错误。     


### 解决途径
最近没有时间来做这个事情，所以只能先写下自己的思路。    

1. 用一个registry缓存所有的包(去掉docker
   load的环节），直接从cache里取回gcr.io的包。docker在启动的时候从cache里取东西回来。但是我不是很确定gcr.io是否可以像registry-1.docker.io一样被缓存下来。    

2.
伪造gcr.io的签名，指向内部的registry仓库。这个可以仿照我的kubespray框架中的做法，骗过dind。    


离线情况下玩docker，真是没事找事，一波三折啊！！！     
