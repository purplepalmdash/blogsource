+++
title = "GitLabCIForK8s"
date = "2018-09-12T09:53:15+08:00"
description = "GitLabCIForK8s"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Environment
中心节点: ` zzz_gitlabci_center`,
虚拟机，4核3G内存，200G硬盘，接入到`192.168.122.0/24`的kvm default网络，
上面部署portus, kubespray, nfs等k8s部署组件，而后部署gitlab环境。     

k8s工作节点 `zzz_gitlabci_allinone_k8s`,    
虚拟机， 4核6G内存，200G硬盘。    

### gitlab bug
GitLab reconfigure:    

```
# vim /etc/gitlab/gitlab.rb
external_url 'http://192.168.122.171'
# vim /opt/gitlab/embedded/service/gitlab-rails/lib/gitlab/kubernetes/helm/base_command.rb
-            apk add -U ca-certificates openssl >/dev/null
+            apk add -U wget ca-certificates openssl >/dev/null
# cat > /etc/default/locale <<EOF
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
LANGUAGE=en_US.UTF-8
EOF
# localedef -v -c -i en_US -f UTF-8 en_US.UTF-8 || true
# gitlab-ctl reconfigure && gitlab-ctl restart
```

### portus bug
原有的portus管理端口占用了80端口，因而需要手动改动其为81端口，以响应web页面的管理。   
### 配置
参考:    
https://edenmal.moe/post/2018/GitLab-Kubernetes-Using-GitLab-CIs-Kubernetes-Cluster-feature/

初始化的gitlab界面:    

![/images/2018_09_12_11_09_11_749x466.jpg](/images/2018_09_12_11_09_11_749x466.jpg)

创建的项目:    

![/images/2018_09_12_11_09_37_617x548.jpg](/images/2018_09_12_11_09_37_617x548.jpg)

从示例项目创建新项目：    

```
# mkdir Code
# cd Code
# git clone https://github.com/galexrt/presentation-gitlab-k8s.git
# cd presentation-gitlab-k8s/
# git remote remove origin
# git remote add origin git@192.168.122.171:root/gitlabci.git
# git push origin master
```
此时应该可以看到CI/CD按钮处于pending状态:    

![/images/2018_09_12_11_20_04_316x175.jpg](/images/2018_09_12_11_20_04_316x175.jpg)

### 获取TOKEN
登录到`192.168.122.172`	机器，执行以下操作获取token:    

```
# kubectl get secret
NAME                  TYPE                                  DATA      AGE
default-token-spz8w   kubernetes.io/service-account-token   3         58m
# kubectl get secret default-token-spz8w -o yaml | grep token:
xxxxxxxxxxxx
# cat xxxxxxx | base64 -d
```
拷贝得到的decode 后的token字段，后面我们需要在gitlab页面中填入。    
### 获取CA
在`192.168.122.172`机器上，查看当前的kubectl配置:    

```
# kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://192.168.122.172:6443
  name: cluster.local
# cat ~/.kube/config
apiVersion: v1
kind: Config
current-context: admin-cluster.local
preferences: {}
clusters:
- cluster:
    certificate-authority-data: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    server: https://192.168.122.172:6443
  name: cluster.local
# cat xxxxxxxxxxxxxxxxxxxxxxxxxx | base64 -d
```
将输出的内容保存，后面我们将在gitlab配置页面中输入该内容.    

### gitlabci & k8s cluster
点击下述图标进入到k8s cluster的配置:    

![/images/2018_09_12_11_36_37_400x302.jpg](/images/2018_09_12_11_36_37_400x302.jpg)    

添加一个已有的k8s 集群:    

![/images/2018_09_12_11_40_18_615x263.jpg](/images/2018_09_12_11_40_18_615x263.jpg)

填入以下内容:    

![/images/2018_09_12_11_43_55_600x620.jpg](/images/2018_09_12_11_43_55_600x620.jpg)

Applications这里暂时没有安装，需要全程翻墙, 感谢你娘的土共啊:    

![/images/2018_09_12_11_44_55_875x617.jpg](/images/2018_09_12_11_44_55_875x617.jpg)

翻墙后你会遇到这个问题:    

![/images/2018_09_12_11_50_48_861x205.jpg](/images/2018_09_12_11_50_48_861x205.jpg)

```
# kubectl create clusterrolebinding permissive-binding --clusterrole=cluster-admin --user=admin --user=kubelet --group=system:serviceaccounts
```
报api-token无效的时候，返回去干掉namespace, 之后即可继续安装.    


