+++
title = "WorkingTipsOnKismatic"
date = "2018-01-07T14:50:01+08:00"
description = "WorkingTipsOnKismatic"
keywords = ["k8s"]
categories = ["Technology"]
+++
### 网络规划
为了模拟kismatic完全离线安装，我们创建一个完全隔离的网络如下:    

![/images/2018_01_07_15_01_42_420x391.jpg](/images/2018_01_07_15_01_42_420x391.jpg)

![/images/2018_01_07_15_02_13_418x482.jpg](/images/2018_01_07_15_02_13_418x482.jpg)

详细信息如下：     

```
10.15.205.1/24
dhcp: 10.15.205.128 ~ 10.15.205.254
部署节点:  10.15.205.2
etcd01: 
master01:
worker01:
```
说明：
设置dhcp是为了让虚拟机在启动的时候自动获得一个地址，实际上在部署过程中我们都会手动修改节点的
IP地址以与kismatic的配置相匹配。    

### 准备工作
#### CentOS7 Base镜像

- CentOS 7: `CentOS-7-x86_64-Minimal-1708.iso` 最小化安装.    
安装时注意事项： 不要选择swap分区，否则在默认部署kismatic时会提示失败。    
安装完毕后，关闭selinux, 关闭firewalld服务。    
将`kismaticuser.key.pub`注入到系统目录下,
这里的系统目录指的是`/root/.ssh/authorized_keys`，
或者自己用户的`/home/xxxxxx/.ssh/authorized_keys`.    

准备完毕后，关闭此虚拟机，将其虚拟磁盘作为base盘, 用于创建其他节点。    

#### 部署节点(仓库+Registry)
镜像节点是部署成功与否的关键，在这个节点上，我们将创建用于部署kismatic的所有
CentOS仓库镜像，并搭建基于Docker Registry的私有仓库。    

该节点设置为1核cpu，1G内存, IP为`10.15.205.2`。    

```
$ qemu-img create -f qcow2 -b CentOS7_Base/CentOS7Base.qcows2 Deployment.qcow2
Formatting 'Deployment.qcow2', fmt=qcow2 size=214748364800 backing_file=CentOS7_Base/CentOS7Base.qcows2 cluster_size=65536 lazy_refcounts=off refcount_bits=16
$ ls
CentOS7_Base  Deployment.qcow2
```
可以使用`nmtui`来更改其Ip地址/网关等, 注意地址填写为`10.15.205.2/24`.    

![/images/2018_01_07_15_58_01_690x383.jpg](/images/2018_01_07_15_58_01_690x383.jpg)

在主机(10.15.205.1，即我们运行libvirt/kvm的机器)上，从镜像回来的仓库目录下，用python建立一个简单的http服务器，用于初始化安装：    

```
$ ls
base  docker  gluster  kubernetes  updates
$ python2 -m SimpleHTTPServer 8666
```
进入到虚拟机里，更改repo配置:    

```
[root@deployment yum.repos.d]# mkdir back
[root@deployment yum.repos.d]# mv * back
mv: cannot move ‘back’ to a subdirectory of itself, ‘back/back’
[root@deployment yum.repos.d]# vim base.repo
[base]
name=Base
baseurl=http://10.15.205.1:8666/base
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[updates]
name=Updates
baseurl=http://10.15.205.1:8666/updates
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[docker]
name=Docker
baseurl=http://10.15.205.1:8666/docker
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[kubernetes]
name=Kubernetes
baseurl=http://10.15.205.1:8666/kubernetes
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[gluster]
name=gluster
baseurl=http://10.15.205.1:8666/gluster
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[root@deployment yum.repos.d]# yum makecache
```
该台服务器需担任两个角色，镜像服务器和仓库服务器，首先我们来配置仓库服务器:    

```
# yum install yum-utils httpd createrepo
# systemctl enable httpd
# systemctl start httpd
```
host机器上打开`http://10.15.205.2`, 看到以下画面说明仓库服务器安装成功:    

![/images/2018_01_07_16_08_22_727x424.jpg](/images/2018_01_07_16_08_22_727x424.jpg)

建立仓库很简单，参考:    

[https://github.com/apprenda/kismatic/blob/master/docs/disconnected_install.md](https://github.com/apprenda/kismatic/blob/master/docs/disconnected_install.md)    

使用reposync将远端仓库的内容镜像到本地即可。    

例如:    

```
[root@deployment html]# ls
base  docker  gluster  kubernetes  updates
[root@deployment html]# ls base/
Packages  repodata
```
则看到的仓库如下：    

![/images/2018_01_07_16_11_31_472x291.jpg](/images/2018_01_07_16_11_31_472x291.jpg)

接下来开始创建registry仓库，这里有一个bug，就是需要container-selinux-2.21-1.el7.noarch.rpm这个包。
我们手动从网站下载，然后安装之:    

```
# yum install -y container-selinux-2.21-1.el7.noarch.rpm
# yum install -y docker-ce
```
因为我们的服务需要用到`docker-compose`，短时间连通网络并安装`docker-compose`:    

```
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# yum install -y python-pip
# pip install docker-compose
```
预装入本地镜像(以下的命令并不能真正运行，是我自己的批量导入脚本)。    

```
for i in `ls *.tar`
do 
	docker load<$i
	docker tag.....
done
```

我们将参考这篇文章来设置好docker-registry mirror:    

[https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-14-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-14-04)    

配置好以后的文件夹直接就可以迁移到别的机器上，事实上，我的位于Centos7服务器上的目录正是从一台Ubuntu
机器上迁移过来的，注意在配置签名的时候需要指定域名，而后，在需使用该docker registry的机器上需要对应添加`/etc/hosts`中的条目:    

```
# vim /etc/hosts
10.15.205.2 mirror.xxxx.com
# docker login mirror.xxxx.com
Username (clouder): clouder
Password: 
Login Succeeded
```

确认服务可用以后，我们可以使用systemd将docker-compose启动的服务添加为系统服务，这样每次重新
启动机器后，我们的registry服务也将随机器启动而启动:    

```
# vim /etc/systemd/system/docker-compose.service 
[Unit]
Description=DockerCompose
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/bin/docker-compose -f /docker-registry/docker-compose.yml up -d

[Install]
WantedBy=multi-user.target
# systemctl enable docker-compose.service
# systemctl start docker-compose.service
```
#### 节点机准备
需要准备三台节点机，创建如下:    

```
# qemu-img create -f qcow2 -b CentOS7_Base/CentOS7Base.qcows2 etcd01.qcow2
# qemu-img create -f qcow2 -b CentOS7_Base/CentOS7Base.qcows2 master01.qcow2
# qemu-img create -f qcow2 -b CentOS7_Base/CentOS7Base.qcows2 worker01.qcow2
```
可以在已有的虚拟机基础上稍加修改，即可得到新的三台机器:    

```
# sudo virsh dumpxml kismatic_deployment>template.xml
# cp template.xml etcd01.xml
# vim etcd01.xml
# sudo virsh define etcd01.xml
Domain kismatic_etcd01 defined from etcd01.xml
```
进入到系统后，配置仓库，配置好`/etc/hosts`下的条目，节点机即做完准备。    

### 部署
配置过程以后再写。

### 使用集群
集群部署完成后，使用`./kismatic dashboard`来访问kubernetes的dashboard:   

![/images/2018_01_08_16_11_55_550x654.jpg](/images/2018_01_08_16_11_55_550x654.jpg)

根据提示配置好kubeconfig文件即可。

### 镜像仓库使用
要使用镜像仓库作为集群的中心仓库，外围机器（用于上传和管理镜像的机器）需要做以下设置（以DEBIAN为例）：    

```
# mkdir -p /usr/local/share/ca-certificates/docker-dev-cert/
# cp KISMATIC_FOLDER/devdockerCA.crt /usr/local/share/ca-certificates/docker-dev-cert/
# update-ca-certificates
# systemctl restart docker
# echo "10.15.205.113 mirror.xxxx.com">>/etc/hosts
# docker login mirror.xxxxx.com
Username: clouder
Password: 
Login Succeeded
```
Thus you could directly push images to the registry mirror.    
