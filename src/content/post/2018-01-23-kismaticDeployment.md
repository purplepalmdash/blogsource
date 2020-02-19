+++
title = "kismaticDeployment"
date = "2018-01-23T11:27:12+08:00"
description = "kismaticDeployment"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 环境准备
三台机器(可为虚拟机或物理机):    

```
aptly部署节点(部署完毕后可关机): 192.192.189.110
镜像仓库部署节点(部署完毕后可关机): 192.192.189.111
k8s节点机: 192.192.189.125
```

操作系统:
节点机，安装Ubuntu16.04，
这里我们采用Base镜像的方式，即时启动一台虚拟机，首先更改其用户名/密码配置及host文件:    

```
# passwd
输入想输入的密码
# vim /etc/ssh/sshd_config
PermitRootLogin yes
# systemctl restart sshd
# reboot
# vim /etc/apt/sources.list
deb http://192.192.189.110	xenial main
deb http://192.192.189.110	kubernetes-xenial main
deb [arch=amd64] http://192.192.189.110	xenial stable
# cat mykey.asc | apt-key add -
# vim /etc/hosts
192.192.189.111	mirror.xxxxxx.com
``` 
说明： 上述过程中的mykey.asc来自于aptly服务器，
`/etc/apt/sources.list`里的IP地址为aptly服务器的IP地址。 

由上述修改过的基础镜像创建出实际的虚拟机可用镜像:    

```
# qemu-img create -f qcow2 -b Base/Ubuntu1604_base.qcow2 RHPT.qcow2
```
创建一台虚拟机，桥接模式，16核，64G内存：   

![/images/2018_01_23_11_56_06_398x255.jpg](/images/2018_01_23_11_56_06_398x255.jpg)

配置桥接模式:    

![/images/2018_01_23_11_58_23_579x263.jpg](/images/2018_01_23_11_58_23_579x263.jpg)

配置IP地址/子网/网关等:    

![/images/2018_01_23_12_11_43_351x258.jpg](/images/2018_01_23_12_11_43_351x258.jpg)

另外的一些优化配置在这里不详细提到。   

注意: registry-mirror机器是CentOS7环境, 相应镜像中的IP地址修改如图:    

![/images/2018_01_23_12_15_09_537x276.jpg](/images/2018_01_23_12_15_09_537x276.jpg)

### kismatic部署
在原有的配置中，直接拷贝一个出来:    

```
# cp -r ubuntuone RongHePingTai
# cd RongHePingTai
# rm -rf generated
# cp kismatic-cluster.yaml kismatic-cluster.yaml.back
# vim kismatic-cluster.yaml
这里作相应的修改，修改过的文件不详细列举
```

安装过程:    

```
# ./kismatic install apply
```
安装过程大约需要10分钟，耐心等候。

### 配置
####  本地工具配置
配置本地工具helm和kubectl:    

```
# cp kubectl /usr/bin
# mkdir -p ~/.kube
# cp generated/kubeconfig ~/.kube/config
# cp helm /usr/bin/
# kubectl version
```
#### host-path存储配置
在本地的initial/k8s-hostpath-provisioner目录下，上传两个tar文件到k8s节点机，load，
而后执行:    

```
# cd examples
# # kubectl create -f sa.yaml -f cluster-roles.yaml -f cluster-role-bindings.yaml -f ds.yaml
# kubectl get ds -n kube-system | grep hostpath
hostpath-provisioner   0         0         0         0            0           nailgun.name/hostpath=enabled   16s
```
定义节点ubuntu为可以部署hostPath volumes的节点, 并检查label是否被设置成功:    

```
# kubectl get nodes
NAME      STATUS    ROLES     AGE       VERSION
ubuntu    Ready     master    7d        v1.9.0
# kubectl label node ubuntu nailgun.name/hostpath=enabled
node "ubuntu" labeled
# kubectl get nodes ubuntu --show-labels
NAME      STATUS    ROLES     AGE       VERSION   LABELS
ubuntu    Ready     master    7d        v1.9.0    beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kismatic/cni-provider=calico,kismatic/kube-proxy=true,kubernetes.io/hostname=ubuntu,nailgun.name/hostpath=enabled,node-role.kubernetes.io/master=
```
在节点上注明可用的存储：    

```
# kubectl annotate node ubuntu hostpath.nailgun.name/ssd=/data
node "ubuntu" annotated
```

接着创建sc(StorageClass), 并检查之:    

```
# kubectl create -f sc.yaml
storageclass "local-ssd" created
# kubectl get sc
NAME                  PROVISIONER             AGE
local-ssd (default)   nailgun.name/hostpath   10s
```
#### nginx-ingress controller
运行如下命令部署之:    

```
# cd initial/nginx-ingress-images
上传tar文件至k8s工作节点上，手动load之
# cd initial/nginx-ingress
# helm install --name "ingressenter" .
```
运行后截图如下:    

![/images/2018_01_23_14_17_47_857x99.jpg](/images/2018_01_23_14_17_47_857x99.jpg)

#### monocular(可选)
上传镜像，并安装之:    

```
# cd monocular-images
将所有的image上传到k8s节点上，安装之。
```
安装monocular charts:    

```
# cd initial/monocular
# helm install --name "xxxxxxx" .
```
安装完毕后： 

![/images/2018_01_23_14_49_05_822x700.jpg](/images/2018_01_23_14_49_05_822x700.jpg)
