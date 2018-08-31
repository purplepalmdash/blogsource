+++
title = "Kubespray全离线部署"
date = "2018-08-29T09:09:48+08:00"
description = "KubesprayOffline"
keywords = ["Linux"]
categories = ["Linux"]
+++
离线部署方案说起来很简单，做起来比较繁琐，把Internet连上一次部署成功，再断开后部署成功一次，那下次就直接能用了。
### 在线状态
前提条件，全翻墙网络，修改Vagrantfile中的操作镜像版本为centos,
网络接口为calico:    

```
...
$os = "centos"
...
$network_plugin = "calico"
...
```
因为我们用的centos默认是不缓存安装包的，因而在`/etc/yum.conf`中需要手动打开其缓存包目录：    

```
# vim /etc/yum.conf
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=1
```
在线部署一次，只要能成功，那么`/var/cache/yum/`下将缓存所有的rpm包
### 离线部署
拷贝出一个新的离线部署目录，并删除该目录下的.vagrant目录，并修改vagrant的主机名称，否则默认会使用一样的主机名来部署系统, 为避免网络冲突，应该更改离线环境的网段为新网段.    

```
# cp -r kubespray kubespray_centos_offline
# vim Vagrantfile
$instance_name_prefix = "k8s-offline-centos"
$subnet = "172.17.89"
```
断开Internet连接, `vagrant up`设置初始化环境。显然会卡在第一步， yum仓库更新.    

![/images/2018_08_29_09_16_22_765x736.jpg](/images/2018_08_29_09_16_22_765x736.jpg)

#### 离线yum仓库
在一台在线部署成功的机器上运行以下命令以取回包:    

```
# mkfit /home/vagrant/kubespray_pkgs_ubuntu/
# find . | grep rpm$ | xargs -I % cp % /home/vagrant/kubespray_pkgs_ubuntu/
# createrepo_c .
# scp -r kubespray_pkgs_ubuntu root@172.17.89.1:/web-server-folder
```
修改ansible playbook:     

```
# vim ./roles/kubernetes/preinstall/tasks/main.yml
- name: Update package management repo address (YUM)
  shell: mkdir -p /root/repoback && mv /etc/yum.repos.d/*.repo /root/repoback && curl http://172.17.88.1/kubespray_pkgs_ubuntu/kubespray.repo>/etc/yum.repos.d/kubespray.repo

- name: Update package management cache (YUM)
```
继续安装, 会在安装docker处失败。     

#### Docker安装
默认会添加`docker.repo`定义，因为我们在以前已经离线缓存了docker包，这里注释掉:    

```
# vim ./roles/kubernetes/preinstall/tasks/main.yml
    #- name: Configure docker repository on RedHat/CentOS
    #  template:
    #    src: "rh_docker.repo.j2"
    #    dest: "{{ yum_repo_dir }}/docker.repo"
    #  when: ansible_distribution in ["CentOS","RedHat"] and not is_atomic
```
接下来继续安装，会在`Download containers if pull is required or told to always pull (all nodes)`处失败.    

### Docker镜像
offline的情形还没有试出来，暂时禁止自动下载，手动上传到节点。    

```
# vim roles/download/defaults/main.yml
# Used to only evaluate vars from download role
skip_downloads: True
```
在线节点上，保存离线镜像的脚本:     

```
docker save gcr.io/google-containers/hyperkube-amd64:v1.11.2>1.tar
docker save quay.io/calico/node:v3.1.3>2.tar
docker save quay.io/calico/ctl:v3.1.3>3.tar
docker save quay.io/calico/kube-controllers:v3.1.3>4.tar
docker save quay.io/calico/cni:v3.1.3>5.tar
docker save nginx:1.13>6.tar
docker save gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.10>7.tar
docker save gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.10>8.tar
docker save gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.10>9.tar
docker save quay.io/coreos/etcd:v3.2.18>9.tar
docker save gcr.io/google_containers/cluster-proportional-autoscaler-amd64:1.1.2>10.tar
docker save gcr.io/google_containers/pause-amd64:3.0>11.tar
```

### Portus镜像仓库配置
参考:    
https://purplepalmdash.github.io/blog/2018/05/30/synckismaticimages/

创建team:    

![/images/2018_08_29_11_35_00_375x296.jpg](/images/2018_08_29_11_35_00_375x296.jpg)

Admin->User->Create new user, 创建一个名为kubespray的用户:    

![/images/2018_08_29_11_35_51_453x319.jpg](/images/2018_08_29_11_35_51_453x319.jpg)

Team->kubespray, Add memeber:    

![/images/2018_08_29_11_36_48_506x296.jpg](/images/2018_08_29_11_36_48_506x296.jpg)

创建一个新的命名空间kubesprayns，并绑定到kubespray组:    

![/images/2018_08_29_11_38_16_435x349.jpg](/images/2018_08_29_11_38_16_435x349.jpg)

查看Log:    

![/images/2018_08_29_11_38_43_588x302.jpg](/images/2018_08_29_11_38_43_588x302.jpg)

### 同步镜像到仓库
首先登录到我们刚才创建的仓库:    

```
# docker login portus.xxxx.com:5000/kubesprayns
Username: kubespray
Password: xxxxxxx
Login Succeeded
```
加载我们之前离线的镜像， 加tag, push.    

```
# for i in `ls *.tar`; do docker load<$i; done
# ./tag_and_push.sh
```
脚本如下:    

![/images/2018_08_29_11_54_22_948x451.jpg](/images/2018_08_29_11_54_22_948x451.jpg)

之后我们可以获得纯净的`/var/lib/portus`目录用于部署kubespray.    

接下来替换掉原有的编译脚本，编译出新的ISO

TODO： ansible需要安装， rpm包安装.    
### ansible部署
起先用vagrant做的一键部署方案，如今需要手动构建出一个集群的定义文件。    
