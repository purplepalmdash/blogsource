+++
title = "WorkingTipsOnKismaticCentOS"
date = "2018-02-11T09:22:08+08:00"
description = "WorkingtipsOnKismaticCentOS"
keywords = ["Linux"]
categories = ["Technology"]
+++
### 环境准备
IP地址及CentOS版本截图如下:    

![/images/2018_02_11_09_25_08_418x157.jpg](/images/2018_02_11_09_25_08_418x157.jpg)

`nmtui`改动hostname及IP地址:    

![/images/2018_02_11_09_26_12_441x226.jpg](/images/2018_02_11_09_26_12_441x226.jpg)

![/images/2018_02_11_09_26_44_600x238.jpg](/images/2018_02_11_09_26_44_600x238.jpg)

改动完毕以后的系统如下:    

![/images/2018_02_11_09_27_36_429x312.jpg](/images/2018_02_11_09_27_36_429x312.jpg)

### 准备部署
Kubernetes在部署kubelet的时候需要检测swap分区是否为关闭，否则无法继续安装，因而我们
首先关闭掉swap分区:    

```
# swapoff -a
```

关闭默认安装的防火墙firewalld，否则kismatic安装时将在helm安装一节报错:    

```
# systemctl stop firewalld && systemctl disable firewalld
```

配置安装所需的yum仓库, `192.192.189.111`是我们搭建在内网的安装源:    

```
[root@docker-1 ~]# cd /etc/yum.repos.d/
[root@docker-1 yum.repos.d]# ls
CentOS-Base.repo  CentOS-CR.repo  CentOS-Debuginfo.repo  CentOS-fasttrack.repo
CentOS-Media.repo  CentOS-Sources.repo  CentOS-Vault.repo
[root@docker-1 yum.repos.d]# mkdir back
[root@docker-1 yum.repos.d]# mv * back
mv: cannot move ‘back’ to a subdirectory of itself, ‘back/back’
[root@docker-1 yum.repos.d]# vi base.repo
[base]
name=Base
baseurl=http://192.192.189.111/base
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[updates]
name=Updates
baseurl=http://192.192.189.111/updates
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[docker]
name=Docker
baseurl=http://192.192.189.111/docker
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[kubernetes]
name=Kubernetes
baseurl=http://192.192.189.111/kubernetes
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[gluster]
name=gluster
baseurl=http://192.192.189.111/gluster
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[other]
name=other
baseurl=http://192.192.189.111/other
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[root@docker-1 yum.repos.d]# yum makecache
```
配置Kubernetes所需的docker镜像仓库（离线）,我们只需在系统中添加`mirror.dddd.com`
域名解析即可，docker镜像仓库也搭建在`192.192.189.111`机器上：    

```
[root@docker-1 yum.repos.d]# vi /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.192.189.111	mirror.dddd.com
```

### 开始部署
进入`gj_deploy`目录，这里包含了用于部署Kubernetes系统的所有安装定义文件，我们需要
改动的主要有两个，

#### 1. 手动安装ssh pub key
这里的ssh pub key来自于安装目录下的`kismaticuser.key.pub`.   

![/images/2018_02_11_09_53_14_805x140.jpg](/images/2018_02_11_09_53_14_805x140.jpg)

将部署所需的ssh pub key安装到节点机上:    

```
[root@docker-1 ~]# vi /root/.ssh/authorized_keys 


ssh-rsa
AAAAB3NzaC1yc2EAAAADAQABAA
......
```
填入的内容就是上面文件的内容

#### kismatic-cluster.yaml
这里只需要改动一行：    

![/images/2018_02_11_09_54_23_545x96.jpg](/images/2018_02_11_09_54_23_545x96.jpg)

签名文件，改动到当前的绝对路径下。这个签名文件是用于离线部署kubernetes 所需docker镜像
所使用的。

### 部署
部署命令非常简单，  

```
# ./kismatic install apply
```
当看到下列画面时，代表安装成功:    

![/images/2018_02_11_09_56_07_967x373.jpg](/images/2018_02_11_09_56_07_967x373.jpg)


### 访问集群
搭建完毕的kismatic集群可以通过以下方式快捷访问:    

```
# ./kismatic dashboard
```
弹出的画面即为Kubernetes的dashboard:    

![/images/2018_02_11_09_57_42_1055x763.jpg](/images/2018_02_11_09_57_42_1055x763.jpg)

需要使用key来访问，key的位置如下(当前目录的generated目录下):    

![/images/2018_02_11_09_59_00_622x355.jpg](/images/2018_02_11_09_59_00_622x355.jpg)

访问成功：   

![/images/2018_02_11_10_17_54_896x603.jpg](/images/2018_02_11_10_17_54_896x603.jpg)


至此，kubernetes环境搭建成功。    

### Dashboard对外访问
更改运行中dashboard的service：    

```
# ./kubectl --kubeconfig generated/kubeconfig edit svc kubernetes-dashboard -n
kube-system
```
将`type:ClusterIP` 改为 `type:NodePort`.    

外部访问则是通过:    

```
# ./kubectl --kubeconfig generated/kubeconfig get svc kubernetes-dashboard -n
kube-system
```
得到kubernetes-dashboard的NodePort端口即可.    

![./images/2018_02_11_11_35_39_698x721.jpg](./images/2018_02_11_11_35_39_698x721.jpg)    
