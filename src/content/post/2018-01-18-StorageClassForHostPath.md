+++
title = "StoragePassUsingLocalPath"
date = "2018-01-18T09:02:02+08:00"
description = "StoragePassUsingLocalPath"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 目的
搭建monocular的时候，从其mongodb的charts变量定义文件`values.yaml`中，可以看到其对persistence的支持，即对持久化存储的支持：    

```
## Enable persistence using Persistent Volume Claims
## ref: http://kubernetes.io/docs/user-guide/persistent-volumes/
##
persistence:
  enabled: true
  ## mongodb data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  # storageClass: "-"
  accessMode: ReadWriteOnce
  size: 8Gi
```
这个charts在下发的时候，请求了一块大小为8G的持久化存储空间，用于存放mongodb的数据。注释中也列出了支持持久化存储的一些解决方案，例如AWS,
GKE, AWS和OpenStack。    

### 问题
通过helm安装这个包的时候，在minikube上安装成功，在自己搭建的环境上，却只能通过如下命令安装成功:    

```
# helm install --name=tiger --set "persistence.enabled=false,mongodb.persistence.enabled=false,pullPolicy=IfNotPresent,api.image.pullPolicy=IfNotPresent,ui.image.pullPolicy=IfNotPresent,prerender.image.pullPolicy=IfNotPresent" monocular/monocular
```
上面的命令禁用了mongodb的持久化存储选项，并设置了镜像的拉取规则。禁用持久化存储选项后，容器的存储空间在容器生命周期结束以后是无法保持的。为什么在minikube中可以部署的在自己的环境中却不能直接部署呢？

### 调研
#### minikube
minikube使用了自己的hostpath实现，可以通过dashboard查看到，minikube默认的sc实际上是映射到/mnt的某个目录下。   

#### 自己的k8s环境
首先在dashboard中我们并没有发现对应的条目，pv及pvc也没有创建出来。

```
[root@DashSSD dash]# kubectl get pvc
No resources found.
[root@DashSSD dash]# kubectl get pv
No resources found.
[root@DashSSD dash]# kubectl get storageclass
No resources found.
```
### 实现hostpath
在社区里找到的方案，链接如下:    

[https://github.com/nailgun/k8s-hostpath-provisioner](https://github.com/nailgun/k8s-hostpath-provisioner)   

离线情况下安装流程:    

```
# git clone https://github.com/nailgun/k8s-hostpath-provisioner
# sudo docker pull gcr.io/google_containers/busybox:1.24 && sudo docker save gcr.io/google_containers/busybox:1.24>busybox.tar
# sudo docker pull nailgun/k8s-hostpath-provisioner:latest && sudo docker save nailgun/k8s-hostpath-provisioner:latest>k8shostpath.tar
``` 
将tar文件和目录传送到离线环境，首先安装镜像。

```
# docker load<k8shostpath.tar && docker load<busybox.tar
8f9199423bb0: Loading layer [==================================================>] 41.73 MB/41.73 MB
Loaded image: nailgun/k8s-hostpath-provisioner:latest
2c84284818d1: Loading layer [==================================================>] 1.312 MB/1.312 MB
5f70bf18a086: Loading layer [==================================================>] 1.024 kB/1.024 kB
Loaded image: gcr.io/google_containers/busybox:1.24
```

默认的example目录下的yaml定义文件我们需要做点修改，才能确保部署正确，首先修改ds.yaml，这个文件主要用于定义DaemonSet，
我们需要对里面的image拉取原则进行定义：    

```
      containers:
      - name: hostpath-provisioner
        image: nailgun/k8s-hostpath-provisioner:latest
+++        imagePullPolicy: IfNotPresent
```
接着修改sc.yaml，这里定义的是StorageClass, 设置为默认:    

```
metadata:
  name: local-ssd
+++  annotations:
+++    storageclass.kubernetes.io/is-default-class: "true"
```

创建本地存储目录，并定义为默认的storage class:    

```
# kubectl create -f sa.yaml -f cluster-roles.yaml -f cluster-role-bindings.yaml -f ds.yaml
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
如果想测试pvc/pv，则:    

```
# kubectl create -f pvc.yaml -f test-pod.yaml
```
结果可以通过登录节点机器ubuntu来查看:    

```
root@ubuntu:/data# ls
pvc-409fc19d-fbf3-11e7-b46e-525400e39849
root@ubuntu:/data# ls
pvc-409fc19d-fbf3-11e7-b46e-525400e39849
root@ubuntu:/data# cat pvc-409fc19d-fbf3-11e7-b46e-525400e39849/test 
hello
```
pod周期完毕后，pv/pvc的定义如下:    

```
# kubectl get pvc
NAME        STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
local-ssd   Bound     pvc-409fc19d-fbf3-11e7-b46e-525400e39849   1Mi        RWX            local-ssd      2m
# kubectl get pv
NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM               STORAGECLASS   REASON    AGE
pvc-409fc19d-fbf3-11e7-b46e-525400e39849   1Mi        RWX            Delete           Bound     default/local-ssd   local-ssd                3m
```

重启之后查看pv/pvc/sc，均保留在磁盘上。
