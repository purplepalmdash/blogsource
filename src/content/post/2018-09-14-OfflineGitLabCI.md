+++
title = "全离线环境下实现GitLabCI与Kubernetes对接"
date = "2018-09-14T14:14:58+08:00"
description = "OfflineGitLabCIAndK8s"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 环境准备
本文将构建一个包含4台工作机器的集群来模拟全离线环境下的GitLabCI环境及Kubernetes集群环境。相应的硬件配置如下:    

虚拟机`zz_z_gitlab`:    

```
8 Core, 16G, 200G Disk
gitlab IP: 192.168.122.160/24
```
虚拟机`zz_z_gitlab_runner1`, `zz_z_gitlab_runner2`, `zz_z_gitlab_runner3`:    

```
8 Core, 16G, 200G Disk
runner1 IP: 192.168.122.161/24
runner2 IP: 192.168.122.162/24
runner3 IP: 192.168.122.163/24
```
说明:    
gitlab比较费内存，因而节点分配的计算资源相对来说需要高一点。    
以上的配置是在服务器上，因而没有考虑到最小资源分配问题，在个人电脑上，因为内存和CPU的限制，可以使用2核4G的节点用来搭建Gitlab，2核4G用来搭建k8s单节点集群做实验。    

操作系统：采用Ubuntu16.04作为环境的推荐系统。在其上部署KubeSpray部署的Kubernetes集群。操作系统的安装及Kubernetes的搭建并不在本文范畴内。    

Kubernetes环境截图:    

![/images/2018_09_14_15_19_50_568x136.jpg](/images/2018_09_14_15_19_50_568x136.jpg)

### Registry准备(可选)
离线情况下，需要事先将Kubernetes用到的容器镜像，以及本文gitlabci中用到的镜像全部离线放入到私有仓库，本文中采用的容器镜像私有仓库方案是opensuse提供的Portus，
以容器的方式运行于`zz_z_gitlab`节点上，当然你也可以使用gitlab自带的registry服务来实现， 或者使用外置的harbor等方式。    

Portus与GitLab存在端口重复的问题，需要在Portus的配置文件中，将默认监听的80端口改为其他端口，如81端口。    
### 离线安装文件准备
需要实现下载gitlab的安装文件(deb格式):    

```
 gitlab-ce_11.0.3-ce.0_amd64.deb
 gitlab-runner_11.0.2_amd64.deb
```
安装gitlab-runner时需要用到helm v2.7.0文件:    

```
# wget https://kubernetes-helm.storage.googleapis.com/helm-v2.7.0-linux-amd64.tar.gz
```

### 离线容器准备
gitlab机器上，添加docker的insecure-registries方式:     

```
# vim  /etc/docker/daemon.json
    {
        "insecure-registries" : ["portus.ooooooo.com:5000"]
    }
# systemctl restart docker
# docker login http://portus.ooooooo.com:5000
Username: kubespray
Password: 
```
alpine:3.6镜像:    

```
# docker tag alpine:3.6 portus.ooooooo.com:5000/kubesprayns/alpine:3.6
# docker push portus.ooooooo.com:5000/kubesprayns/alpine:3.6
```
![/images/2018_09_14_15_58_13_435x155.jpg](/images/2018_09_14_15_58_13_435x155.jpg)

用于install helm的alpine镜像准备:    

```
$ sudo docker run -it alpine:3.6 /bin/sh
/ # apk add -U wget ca-certificates openssl >/dev/null
/ # which wget
/usr/bin/wget
/ # ls -l -h /usr/bin/wget
-rwxr-xr-x    1 root     root      443.2K May 11 13:02 /usr/bin/wget
```
保存容器，并上传:    

```
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
79923690bcc2        alpine:3.6          "/bin/sh"           About a minute ago   Up About a minute                       jolly_banach
$ sudo docker commit 79923690bcc2 alpinewithwget:3.6
# docker tag alpinewithwget:3.6 portus.ooooooo.com:5000/kubesprayns/alpinewithwget:3.6
# docker push portus.ooooooo.com:5000/kubesprayns/alpinewithwget:3.6
```
helm在初始化的时候需要用到

```
$ sudo docker pull gcr.io/kubernetes-helm/tiller:v2.7.0
$ sudo docker tag gcr.io/kubernetes-helm/tiller:v2.7.0  portus.ooooooo.com:5000/kubesprayns/gcr.io/kubernetes-helm/tiller:v2.7.0 
$ sudo docker push portus.ooooooo.com:5000/kubesprayns/gcr.io/kubernetes-helm/tiller:v2.7.0 
```
gitlab-runner所需镜像:    

```
$ sudo docker pull busybox:latest
$ sudo docker pull ubuntu:16.04
$ sudo docker pull gitlab/gitlab-runner:alpine-v10.3.0
$ sudo docker pull gitlab/gitlab-runner-helper:x86_64-latest
$ sudo docker tag busybox:latest portus.ooooooo.com:5000/kubesprayns/busybox:latest
$ sudo docker tag ubuntu:16.04 portus.ooooooo.com:5000/kubesprayns/ubuntu:16.04
$ sudo docker tag gitlab/gitlab-runner:alpine-v10.3.0 portus.ooooooo.com:5000/kubesprayns/gitlab/gitlab-runner:alpine-v10.3.0
$ sudo docker tag gitlab/gitlab-runner-helper:x86_64-latest portus.ooooooo.com:5000/kubesprayns/gitlab/gitlab-runner-helper:x86_64-latest
$ sudo docker push portus.ooooooo.com:5000/kubesprayns/busybox:latest
$ sudo docker push portus.ooooooo.com:5000/kubesprayns/ubuntu:16.04
$ sudo docker push portus.ooooooo.com:5000/kubesprayns/gitlab/gitlab-runner:alpine-v10.3.0
$ sudo docker push portus.ooooooo.com:5000/kubesprayns/gitlab/gitlab-runner-helper:x86_64-latest
```
nginx-ingress-controller所需镜像:    

```
# docker pull quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0
# docker pull k8s.gcr.io/defaultbackend:1.4
# docker tag quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0 portus.ooooooo.com:5000/kubesprayns/quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0
# docker push portus.ooooooo.com:5000/kubesprayns/quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.19.0
# docker tag k8s.gcr.io/defaultbackend:1.4 portus.ooooooo.com:5000/kubesprayns/k8s.gcr.io/defaultbackend:1.4
# docker push portus.ooooooo.com:5000/kubesprayns/k8s.gcr.io/defaultbackend:1.4
```
Prometheus所需容器：    

```
$ sudo docker pull prom/alertmanager:v0.14.0
$ sudo docker pull jimmidyson/configmap-reload:v0.1
$ sudo docker pull quay.io/coreos/kube-state-metrics:v1.3.1
$ sudo docker pull prom/node-exporter:v0.15.2
$ sudo docker pull prom/prometheus:v2.2.1
$ sudo docker pull prom/prometheus:v2.1.0
$ sudo docker pull prom/pushgateway:v0.5.1
$ sudo docker pull prom/alertmanager:v0.14.0
$ sudo docker tag prom/alertmanager:v0.14.0 portus.oooooo:5000/kubesprayns/prom/alertmanager:v0.14.0
$ sudo docker tag jimmidyson/configmap-reload:v0.1 portus.oooooo:5000/kubesprayns/jimmidyson/configmap-reload:v0.1
$ sudo docker tag quay.io/coreos/kube-state-metrics:v1.3.1 portus.oooooo:5000/kubesprayns/quay.io/coreos/kube-state-metrics:v1.3.1
$ sudo docker tag prom/node-exporter:v0.15.2 portus.oooooo:5000/kubesprayns/prom/node-exporter:v0.15.2
$ sudo docker tag prom/prometheus:v2.2.1 portus.oooooo:5000/kubesprayns/prom/prometheus:v2.2.1
$ sudo docker tag prom/prometheus:v2.1.0 portus.oooooo:5000/kubesprayns/prom/prometheus:v2.1.0
$ sudo docker tag prom/pushgateway:v0.5.1 portus.oooooo:5000/kubesprayns/prom/pushgateway:v0.5.1
$ sudo docker tag prom/alertmanager:v0.14.0 portus.oooooo:5000/kubesprayns/prom/alertmanager:v0.14.0
$ sudo docker push  portus.oooooo:5000/kubesprayns/prom/alertmanager:v0.14.0
$ sudo docker push  portus.oooooo:5000/kubesprayns/jimmidyson/configmap-reload:v0.1
$ sudo docker push  portus.oooooo:5000/kubesprayns/quay.io/coreos/kube-state-metrics:v1.3.1
$ sudo docker push  portus.oooooo:5000/kubesprayns/prom/node-exporter:v0.15.2
$ sudo docker push  portus.oooooo:5000/kubesprayns/prom/prometheus:v2.2.1
$ sudo docker push  portus.oooooo:5000/kubesprayns/prom/pushgateway:v0.5.1
$ sudo docker push  portus.oooooo:5000/kubesprayns/prom/alertmanager:v0.14.0
```

项目CI/CD所需容器(test环节/build环节):    

```
# docker pull golang:1.10.3-stretch
# docker tag golang:1.10.3-stretch portus.ooooooo.com:5000/kubesprayns/golang:1.10.3-stretch
# docker push portus.ooooooo.com:5000/kubesprayns/golang:1.10.3-stretch
```
Release阶段所需容器:    

```
# docker pull docker:latest
# docker pull lordgaav/dind-options:latest
# docker tag docker:latest portus.ooooooo.com:5000/kubesprayns/docker:latest
# docker push portus.ooooooo.com:5000/kubesprayns/docker:latest
# docker tag lordgaav/dind-options:latest portus.ooooooo.com:5000/lordgaav/dind-options:latest
# docker push portus.ooooooo.com:5000/kubesprayns/lordgaav/dind-options:latest
```
Deploy阶段所需容器:    

```
# docker pull lachlanevenson/k8s-kubectl:latest
# docker tag lachlanevenson/k8s-kubectl:latest portus.ooooooo.com:5000/kubesprayns/lachlanevenson/k8s-kubectl:latest
# docker push portus.ooooooo.com:5000/kubesprayns/lachlanevenson/k8s-kubectl:latest
```
Build 容器所需要的:    

```
# docker pull busybox:1.28.4-glibc
# docker tag busybox:1.28.4-glibc portus.ooooooo.com:5000/kubesprayns/busybox:1.28.4-glibc
# docker push portus.ooooooo.com:5000/kubesprayns/busybox:1.28.4-glibc
```

### 离线charts准备
下载并更改charts, 而后上传到私有仓库,

#### gitlab-runner
下面是gitlab-runner的charts本地化制作过程:    

```
$ helm repo add runner https://charts.gitlab.io
$ helm fetch runner/gitlab-runner
$ tar xzvf gitlab-runner-0.1.33.tgz
$ cd gitlab-runner
$ vim values.yaml
- gitlab/gitlab-runner:alpine-v10.3.0
+ image: portus.ooooooo.com:5000/kubesprayns/gitlab/gitlab-runner:alpine-v10.3.0

init:
-  image: busybox
+  image: portus.ooooooo.com:5000/kubesprayns/busybox

-  image: ubuntu:16.04
+  image: portus.ooooooo.com:5000/kubesprayns/ubuntu:16.04

-  helpers: {}
+  helpers:

-    # image: gitlab/gitlab-runner-helper:x86_64-latest
+    image: portus.ooooooo.com:5000/kubesprayns/gitlab/gitlab-runner-helper:x86_64-latest
$ helm package .
$ curl --data-binary "@gitlab-runner-0.1.33.tgz" http://portus.ooooooo.com:8988/api/charts
```
#### nginx-ingress离线charts
步骤:    

```
# helm fetch stable/nginx-ingress
# tar xzvf nginx-ingress-0.28.2.tgz
# cd nginx-ingress
# vim values.yaml

<     repository: quay.io/kubernetes-ingress-controller/nginx-ingress-controller
---
>     repository: portus.ooooooo.com:5000/kubesprayns/quay.io/kubernetes-ingress-controller/nginx-ingress-controller
293c293
<     repository: k8s.gcr.io/defaultbackend
---
>     repository: portus.ooooooo.com:5000/kubesprayns/k8s.gcr.io/defaultbackend
# helm package .
$ curl --data-binary "@nginx-ingress-0.28.2.tgz" http://portus.ooooooo.com:8988/api/charts
```
需要配置默认的nginx-ingress-controller的配置:    

```
# vim /opt/gitlab/embedded/service/gitlab-rails/vendor/ingress/values.yaml
- repository: "quay.io/kubernetes-ingress-controller/nginx-ingress-controller"
+ repository: "portus.oooooo.com:5000/kubesprayns/quay.io/kubernetes-ingress-controller/nginx-ingress-controller"
# gitlab-ctl reconfigure && gitlab-ctl restart
```

#### Prometheus离线charts
步骤:    

```
$ helm fetch stable/prometheus --version 6.7.3
$ tar xzvf prometheus-6.7.3.tgz
$ cd prometheus/
$ vim values
35c35
<     repository: prom/alertmanager
---
>     repository: portus.oooooo.com:5000/kubesprayns/prom/alertmanager
212c212
<     repository: jimmidyson/configmap-reload
---
>     repository: portus.oooooo.com:5000/kubesprayns/jimmidyson/configmap-reload
247c247
<     repository: busybox
---
>     repository: portus.oooooo.com:5000/kubesprayns/busybox
268c268
<     repository: quay.io/coreos/kube-state-metrics
---
>     repository: portus.oooooo.com:5000/kubesprayns/quay.io/coreos/kube-state-metrics
345c345
<     repository: prom/node-exporter
---
>     repository: portus.oooooo.com:5000/kubesprayns/prom/node-exporter
439c439
<     repository: prom/prometheus
---
>     repository: portus.oooooo.com:5000/kubesprayns/prom/prometheus
653c653
<     repository: prom/pushgateway
---
>     repository: portus.oooooo.com:5000/kubesprayns/prom/pushgateway
$ helm package .
$ curl --data-binary "@prometheus-6.7.3.tgz" http://portus.ooooooo.com:8988/api/charts
```

### 测试项目准备
测试项目来自于github上的项目:    

```
$ git clone https://github.com/galexrt/presentation-gitlab-k8s.git
```

### 搭建/配置GitLab
本文写作时用到的GitLab版本为`GitLab Community Edition 11.0.3 aa62075`, 供参考.    

```
# dpkg -i gitlab-ce_11.0.3-ce.0_amd64.deb
# dpkg -i gitlab-runner_11.0.2_amd64.deb
```
内网环境下为了避免网络问题，关闭ufw防火墙:    

```
# ufw disable
```
(可选)GitLab默认使用UTF-8，如果配置时碰到编码问题，可在配置完本地编码后，重新登录终端执行reconfigure.    

```
# cat > /etc/default/locale <<EOF
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
LANGUAGE=en_US.UTF-8
EOF
# localedef -v -c -i en_US -f UTF-8 en_US.UTF-8 || true
```
配置gitlab监听的IP端口, 并执行reconfigure, 此后gitlab服务将变得可用:    

```
# vim /etc/gitlab/gitlab.rb
external_url 'http://192.168.122.160'
# gitlab-ctl reconfigure && gitlab-ctl restart
```
配置完成后设置用户名/密码， 搭建成功后的GitLab服务如下:    

![/images/2018_09_14_14_59_27_647x462.jpg](/images/2018_09_14_14_59_27_647x462.jpg)
导入项目:    

![/images/2018_09_14_15_22_45_723x548.jpg](/images/2018_09_14_15_22_45_723x548.jpg)

在准备好的项目中，添加刚才在 Gitlab中创建的项目并提交:    

```
# git remote remove origin
# git remote add origin git@192.168.122.160:root/testk8sci.git
# git push origin master
```
此时可以看到，代码已经被提交到GitLab仓库中，然而ci/cd处于pending状态，接下来将对代码进行修改，将其一步步调试通。    

![/images/2018_09_14_15_26_00_712x290.jpg](/images/2018_09_14_15_26_00_712x290.jpg)

### Gitlab Runner配置
首先需要添加kubernetes集群，而后在集群上安装Helm Tiller， 安装完Helm
Tiller后才可以继续安装GitLab Runner。 选取的项目需要使用GitLab
Runner才可以跑通CI/CD.    

#### 添加Kubernetes集群
点击项目-> Operations->Kubernetes, 如下图:    

![/images/2018_09_14_15_28_57_368x159.jpg](/images/2018_09_14_15_28_57_368x159.jpg)

点击`Add Kubernetes Cluster`按钮，添加一个新的Kubernetes集群:    

![/images/2018_09_14_15_29_28_474x420.jpg](/images/2018_09_14_15_29_28_474x420.jpg)

此时有两个选项，因为是离线的Kubernetes集群，点击`Add an existing Kubernetes
cluster`，继续进入到下一步:    

![/images/2018_09_14_15_30_23_638x218.jpg](/images/2018_09_14_15_30_23_638x218.jpg)

事先需要获得Kubernetes集群的管理信息，通过以下步骤获得，    

获得API Server的地址(runner1节点上):    

```
# kubectl config view | grep server
	server: https://192.168.122.161:6443
```
获得CA证书:    

```
# cat ~/.kube/config | grep certificate-authority-data: 
    certificate-authority-data: XXXXXXXXXXXXXXXXXXXXXXXXXXXX
# cat xxxxxxxxxxxxxxxxxxxxxxxxxx | base64 -d
-----BEGIN CERTIFICATE-----
xxxxxxxxx
-----END CERTIFICATE-----
```
获得Token:    

```
# kubectl get secret
NAME                  TYPE                                  DATA      AGE
default-token-spz8w   kubernetes.io/service-account-token   3         58m
# kubectl get secret default-token-spz8w -o yaml | grep token:
xxxxxxxxxxxx
# cat xxxxxxx | base64 -d
```
将上面获得的各个项目填入到配置项目中:    

![/images/2018_09_14_15_39_40_574x633.jpg](/images/2018_09_14_15_39_40_574x633.jpg)

#### 安装Helm Tiller
安装界面如下:    

![/images/2018_09_14_15_41_52_895x160.jpg](/images/2018_09_14_15_41_52_895x160.jpg)

直接点击`Install`按钮, 出现的问题为:    

```
Kubernetes error: namespaces "gitlab-managed-apps" is forbidden: User
"system:serviceaccount:default:default" cannot get namespaces in the namespace
"gitlab-managed-apps"
```
![/images/2018_09_14_15_43_13_885x233.jpg](/images/2018_09_14_15_43_13_885x233.jpg)

搜索网上的解决方案， 

```
# kubectl create clusterrolebinding permissive-binding --clusterrole=cluster-admin --user=admin --user=kubelet --group=system:serviceaccounts
```
按F5刷新页面并点击`Install`重新安装，安装会一直处于pause，最终将失败，更改以下配置以便继续安装:    

![/images/2018_09_14_16_13_12_696x175.jpg](/images/2018_09_14_16_13_12_696x175.jpg)

```
# vim /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/kubernetes/helm/pod.rb +28
-            image: 'alpine:3.6',
+            image: 'portus.ooooooo.com:5000/kubesprayns/alpinewithwget:3.6',
# vim /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/kubernetes/helm/base_command.rb
-            ALPINE_VERSION=$(cat /etc/alpine-release | cut -d '.' -f 1,2)
-            echo http://mirror.clarkson.edu/alpine/v$ALPINE_VERSION/main >> /etc/apk/repositories
-            echo http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/v$ALPINE_VERSION/main >> /etc/apk/repositories
-            apk add -U wget ca-certificates openssl >/dev/null
-            wget -q -O - https://kubernetes-helm.storage.googleapis.com/helm-v#{Gitlab::Kubernetes::Helm::HELM_VERSION}-linux-amd64.tar.gz | tar zxC /tmp >/dev/null
+            wget -q -O - http://portus.ooooooo.com:8888/helm-v2.7.0-linux-amd64.tar.gz | tar zxC /tmp >/dev/null
```
更改完毕后，我们重新配置gitlab以继续安装:    

```
# gitlab-ctl reconfigure && gitlab-ctl restart
```
此时会出现helm在初始化的时候因为访问不到外网出现的timeout错误，如下:    

![/images/2018_09_14_16_54_54_861x270.jpg](/images/2018_09_14_16_54_54_861x270.jpg)
更改helm init的初始化方式:    

```
# vim /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/kubernetes/helm/install_command.rb
-          'helm init --client-only >/dev/null'
+          'helm init --stable-repo-url http://portus.ooooooo.com:8988 --tiller-image portus.ooooooo.com:5000/kubesprayns/gcr.io/kubernetes-helm/tiller:v2.7.0 >/dev/null'
# /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/kubernetes/helm/init_command.rb
-          "helm init >/dev/null"
+          'helm init --stable-repo-url http://portus.ooooooo.com:8988 --tiller-image portus.ooooooo.com:5000/kubesprayns/gcr.io/kubernetes-helm/tiller:v2.7.0 >/dev/null'

```
更改完毕后，我们重新配置gitlab以继续安装:    

```
# gitlab-ctl reconfigure && gitlab-ctl restart
```
此时终于可以安装成功helm:    

![/images/2018_09_14_17_22_03_876x541.jpg](/images/2018_09_14_17_22_03_876x541.jpg)

#### gitlab-runner
默认点击安装将出现以下错误，:    

![/images/2018_09_14_17_46_58_890x209.jpg](/images/2018_09_14_17_46_58_890x209.jpg)

```
$ vim /opt/gitlab/embedded/service/gitlab-rails/app/models/clusters/applications/runner.rb
-        'https://charts.gitlab.io'
+        'http://portus.ooooooo.com:8988'
```
再次运行，提示configmap已经存在:    

![/images/2018_09_14_17_50_42_866x165.jpg](/images/2018_09_14_17_50_42_866x165.jpg)

删除configmap， 继续运行:    

```
# kubectl delete configmap values-content-configuration-runner -n gitlab-managed-apps
```
至此，我们的gitlab-runner也安装成功:    

![/images/2018_09_14_17_51_39_888x112.jpg](/images/2018_09_14_17_51_39_888x112.jpg)

#### Ingress
默认情况下安装将触发以下错误:     

![/images/2018_09_17_11_08_37_765x333.jpg](/images/2018_09_17_11_08_37_765x333.jpg)
上传完对应的charts文件以后，继续安装:    

![/images/2018_09_17_11_54_52_743x129.jpg](/images/2018_09_17_11_54_52_743x129.jpg)

删除configmaps:    

```
# kubectl delete configmap values-content-configuration-ingress -n gitlab-managed-apps
```
安装成功后:    

![/images/2018_09_17_11_56_04_861x266.jpg](/images/2018_09_17_11_56_04_861x266.jpg)
#### Prometheus
与上述步骤相似，安装完毕后:    

![/images/2018_09_17_14_22_34_859x85.jpg](/images/2018_09_17_14_22_34_859x85.jpg)




### CI/CD项目
此时提交已经可以触发runner，    

![/images/2018_09_17_09_08_28_736x184.jpg](/images/2018_09_17_09_08_28_736x184.jpg)

针对不同的Stage进行Debug. 最终跑通编译。    


