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

### 更新
最近没心思做别的事情，还是把这个PWK的离线给做了，记录一下步骤：    

更新到新的play-with-kubernetes:    

```
# git clone https://github.com/play-with-docker/play-with-kubernetes.github.io
```
当然在这里我们要做适配，以允许其离线化 。    

采用的franela/k8s版本大约是2018年年底的版本，    

```
# franela/k8s              latest              c7038cbdbc5d        2 months ago        733MB
```
因为这个容器镜像中的kubeadm版本已经升级到比较新的版本，需要重新下载镜像:    

```
k8s.gcr.io/kube-proxy-amd64                v1.11.7             e9a1134ab5aa        4 weeks ago         98.1MB
k8s.gcr.io/kube-apiserver-amd64            v1.11.7             d82b2643a56a        4 weeks ago         187MB
k8s.gcr.io/kube-controller-manager-amd64   v1.11.7             93fb4304c50c        4 weeks ago         155MB
k8s.gcr.io/kube-scheduler-amd64            v1.11.7             52ea1e0a3e60        4 weeks ago         56.9MB
weaveworks/weave-npc                       2.5.1               789b7f496034        4 weeks ago         49.6MB
weaveworks/weave-kube                      2.5.1               1f394ae9e226        4 weeks ago         148MB
k8s.gcr.io/coredns                         1.1.3               b3b94275d97c        9 months ago        45.6MB
k8s.gcr.io/etcd-amd64                      3.2.18              b8df3b177be2        10 months ago       219MB
k8s.gcr.io/pause                           3.1                 da86e6ba6ca1        14 months ago       742kB
nginx                                      latest              05a60462f8ba        2 years ago         181MB
```
同时需要更改kubeadm init的参数为:    

```
# kubeadm init --apiserver-advertise-address $(hostname -i) --kubernetes-version=v1.11.7
```
在多节点章节中，kubeaadm
1.11.7与原先的calico3.1冲突，因而我们更新到了更新的3.5版本,
因为docker-in-docker配备了两个网络，我们在calico.yaml中也需要指定IP范围，以确保BGP隧道建立在正确的网络接口上:   

```
            - name: CALICO_IPV4POOL_CIDR
              value: "192.168.0.0/16"
            - name: IP_AUTODETECTION_METHOD
              value: "interface=eth0.*"

```

到此，则新版本的playwithk8s更新完毕，总共花了5天时间。虽然有点磨人，但想起来这5天还是值得的。
