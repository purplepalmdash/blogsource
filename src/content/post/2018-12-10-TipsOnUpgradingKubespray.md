+++
title = "TipsOnKubesprayUpgrading"
date = "2018-12-10T12:08:13+08:00"
description = "TipsOnKubesprayUpgrading"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 目的
Rong部署框架随kubespray社区升级策略。    

### 前提
下载kubespray升级包，如v2.8.0:    

![/images/2018_12_10_12_09_29_628x576.jpg](/images/2018_12_10_12_09_29_628x576.jpg)

将此包解压到某目录，这里解压到:    

```
# cd /var1/myimages/RongUpgrade/280
# ls
kubespray-2.8.0
```

#### 快速同步镜像基准制作
设置一个Rong CentOS7.5 ISO安装后的基础环境, (Base/Rong_Base.qcow2).
制作方法如下（如果已经制作完毕，则可忽略下面步骤):    

激活用于存储cache的rpm包:    

```
# vim /etc/yum.conf
keepcache=1
```
改回原有的仓库配置:    

```
cd /etc/yum.repos.d
mv back/* .
mv kubespray_centos7.repo  back/
```
安装必要的包用于GFW：    

```
yum install -y gcc  libevent-devel
redsocks配置，这里就不说了
```

保存为/var1/myimages/RongUpgrade/Base/Rong_Base.qcow2 ,
以后的每次升级都使用该源包，现在创建一个用于升级2.8.0的镜像:     


```
$ qemu-img create -f qcow2 -b Base/Rong_Base.qcow2 v280.qcow2
```
6核7G的虚拟机配置，选择default网络, 注意更改MAC地址为与上面定义时一致:    

![/images/2018_12_10_12_27_06_497x283.jpg](/images/2018_12_10_12_27_06_497x283.jpg)


#### 获得包/镜像
配置一个用于部署的环境:    

```
# cp deploy.key .
# cp -r inventory/sample inventory/rong
# vim inventory/rong/hosts.ini
[all]
allinone ansible_host=192.168.122.166 ansible_ssh_user=root ansible_ssh_private_key_file=./deploy.key

[kube-deploy]
allinone

[kube-master]
allinone

[etcd]
allinone

[kube-node]
allinone

[k8s-cluster:children]
kube-master
kube-node
```
部署一次:    

```
# ansible-playbook -i inventory/rong/hosts.ini cluster.yml
```
在全FQ的环境下，部署应该可以成功，此时备份容器镜像:    

```
# docker images
.....
# docker images | sed -n '1!p' | awk {'print $1":"$2'}
# docker save -o docker.tar gcr.io/google-containers/kube-proxy:v1.12.3 gcr.io/google-containers/kube-apiserver:v1.12.3 gcr.io/google-containers/kube-controller-manager:v1.12.3 gcr.io/google-containers/kube-scheduler:v1.12.3 coredns/coredns:1.2.6 gcr.io/google_containers/cluster-proportional-autoscaler-amd64:1.3.0 gcr.io/google-containers/coredns:1.2.2 gcr.io/google_containers/kubernetes-dashboard-amd64:v1.10.0 quay.io/coreos/etcd:v3.2.24 quay.io/calico/node:v3.1.3 quay.io/calico/ctl:v3.1.3 quay.io/calico/kube-controllers:v3.1.3 quay.io/calico/cni:v3.1.3 nginx:1.13 gcr.io/google-containers/pause:3.1 gcr.io/google_containers/pause-amd64:3.1
# cp docker.tar.gz /mnt
```
拷贝出的`docker.tar`即含有所有的容器镜像.    

备份安装包,rpm包:    

```
# yum install -y nethogs
# cd /var/cache/
# mkdir -p /root/rpms
# find . | grep rpm$ | xargs -I % cp % /root/rpms/
# cd /root/rpms/
# createrepo .
# cp rpms.tar.gz /mnt
```


#### kubespray源码更改

更改配置脚本:    

```
# cp ~/roles/kube-deploy ./roles/kube-deploy
# 替换rpms文件和docker.tar文件
# cd kubespray-2.8.0/roles/kube-deploy/files
# rm -rf kubespray_centos7_rpms
# cp /mnt/rpms ./kubespray_centos7_rpms
# rm -f kubespray_images.tar.xz
# cp /mnt/docker.tar ./kubespray_images.tar
# xz kubespray_images.tar.xz
# vim tag_and_push.sh
更改这里，为我们的docker images名称->私有仓库名称
```
更改download角色:    

```
# vim ./roles/download/defaults/main.yml
    # Download URLs
    #kubeadm_download_url: "https://storage.googleapis.com/kubernetes-release/release/{{ kubeadm_version }}/bin/linux/{{ image_arch }}/kubeadm"
    #hyperkube_download_url: "https://storage.googleapis.com/kubernetes-release/release/{{ kube_version }}/bin/linux/amd64/hyperkube"
    #etcd_download_url: "https://github.com/coreos/etcd/releases/download/{{ etcd_version }}/etcd-{{ etcd_version }}-linux-amd64.tar.gz"
    #cni_download_url: "https://github.com/containernetworking/plugins/releases/download/{{ cni_version }}/cni-plugins-{{ image_arch }}-{{ cni_version }}.tgz"
    kubeadm_download_url: "http://portus.gggg.com:8888/kubeadm"
    hyperkube_download_url: "http://portus.gggg.com:8888/hyperkube"
    etcd_download_url: "https://github.com/coreos/etcd/releases/download/{{ etcd_version }}/etcd-{{ etcd_version }}-linux-amd64.tar.gz"
    cni_download_url: "http://portus.gggg.com:8888/cni-plugins-{{ image_arch }}-{{ cni_version }}.tgz"
```
更改私有镜像:    

```
# vim inventory/rong/group_vars/k8s-cluster/k8s-cluster.yml
得到镜像名称:    

# cat ./roles/download/defaults/main.yml | grep _image_repo:|grep -v kube_image_repo
如: 
etcd_image_repo: "quay.io/coreos/etcd"
flannel_image_repo: "quay.io/coreos/flannel"
flannel_cni_image_repo: "quay.io/coreos/flannel-cni"
calicoctl_image_repo: "quay.io/calico/ctl"
calico_node_image_repo: "quay.io/calico/node"
加前缀:
etcd_image_repo: "xxxxx/quay.io/coreos/etcd"
....
```

更改cluster.yml,省略docker-ce的安装:    

```
- hosts: k8s-cluster:etcd:calico-rr:!kube-deploy
  any_errors_fatal: "{{ any_errors_fatal | default(true) }}"
  gather_facts: false


    - { role: kubernetes/preinstall, tags: preinstall }
    #- { role: "container-engine", tags: "container-engine", when: deploy_container_engine|default(true) }
    - { role: download, tags: download, when: "not skip_downloads" }
  environment: "{{proxy_env}}"


```
更改deploy-centos的配置:    

```
# roles/bootstrap-os/tasks/bootstrap-centos.yml
- name: Configure intranet repository
  shell: mkdir -p /etc/yum.repos.d/back && mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/back; curl http://portus.teligen.com:8888/kubespray_centos7.repo>/etc/y
um.repos.d/kubespray_centos7.repo && yum makecache && systemctl stop firewalld ; systemctl disable firewalld


- name: Install packages requirements for bootstrap
  yum:
.....
```

验证：

![/images/2018_12_10_16_37_24_599x283.jpg](/images/2018_12_10_16_37_24_599x283.jpg)


