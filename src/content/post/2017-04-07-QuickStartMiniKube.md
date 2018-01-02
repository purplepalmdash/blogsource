+++
description = "QuickStartMiniKube"
title = "快速搭建MiniKube开发环境"
date = "2017-04-07T18:08:19+08:00"
categories = ["Virtualization"]
keywords = ["Virtualization"]

+++
### 版本
minikube的版本是`v0.17.1`, 运行于ArchLinux.    

### 镜像准备
感谢万能的防火墙，我们需要手动下载docker镜像到本地:    

```
sudo docker pull  gcr.io/google-containers/kube-addon-manager:v6.3
sudo docker pull gcr.io/google_containers/kubedns-amd64:1.9
sudo docker pull gcr.io/google_containers/kube-dnsmasq-amd64:1.4
sudo docker pull gcr.io/google_containers/exechealthz-amd64:1.2
sudo docker pull  gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1
sudo docker pull gcr.io/google_containers/heapster:v1.2.0
sudo docker pull kubernetes/heapster_influxdb:v0.6
sudo docker pull gcr.io/google_containers/heapster_grafana:v2.6.0-2
sudo docker pull gcr.io/google_containers/pause-amd64:3.0
```
存储docker镜像并打包的命令如下, 这样一个wget就可取下来所有的镜像:    

```
sudo docker save  gcr.io/google-containers/kube-addon-manager:v6.3 | bzip2>~/serve/addonmanagerv63.tar.bz2
sudo docker save gcr.io/google_containers/kubedns-amd64:1.9|bzip2>~/serve/dns19.tar.bz2
sudo docker save gcr.io/google_containers/kube-dnsmasq-amd64:1.4 |bzip2>~/serve/dnsmasq14.tar.bz2
sudo docker save gcr.io/google_containers/exechealthz-amd64:1.2|bzip2>~/serve/exechealthz12.tar.bz2
sudo docker save  gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1|bzip2>~/serve/dashboard151.tar.bz2
sudo docker save gcr.io/google_containers/heapster:v1.2.0|bzip2>~/serve/heapster.tar.bz2
sudo docker save kubernetes/heapster_influxdb:v0.6|bzip2>~/serve/influx.tar.bz2
sudo docker save gcr.io/google_containers/heapster_grafana:v2.6.0-2|bzip2>~/serve/grafana.tar.bz2
sudo docker save gcr.io/google_containers/pause-amd64:3.0|bzip2>~/serve/pauseamd64.tar.bz2
sudo tar czvf ~/serve/kube.tar.gz ~/serve/*.tar.bz2
```
下载到本地后，直接用`sudo docker load<xxxx.tar.bz2`即可安装这些镜像。    

### minikube
安装minikube的命令为:    

```
$ yaourt minikube
```
启动的命令为:    

```
# minikube stop && minikube start
```
