+++
title= "WorkingTipsOnRongV7.0"
date = "2021-01-20T11:06:18+08:00"
description = "WorkingTipsOnRongV7.0"
keywords = ["Technology"]
categories = ["Technology"]
+++
用于记录基于kubespray v2.15.0离线化过程。     

### 包准备
Download kubespray v2.15.0 source code via:    

```
# wget https://github.com/kubernetes-sigs/kubespray/archive/v2.15.0.tar.gz
```
使用RongRobots得到离线包以便替换。    

```
$ ls -l -h RobotSon.tar.gz 
-rw-r--r-- 1 dash root 882M Jan 20 15:58 RobotSon.tar.gz
```
准备的目录如下:    

```
# mkdir RobotSon
# tar xzvf RobotSon.tar.gz -C RobotSon/
# ls 
kubespray-2.15.0.tar.gz  Origin  RobotSon  RobotSon.tar.gz  Rong

```
### 代码修改
替换静态文件:     

```
# rm -f Rong/pre-rong/rong_static/for_cluster/calicoctl 
# rm -f Rong/pre-rong/rong_static/for_cluster/cni-plugins-linux-amd64-v0.8.7.tgz 
# rm -f Rong/pre-rong/rong_static/for_cluster/kube*
# cp RobotSon/release/calicoctl Rong/pre-rong/rong_static/for_cluster/
# cp RobotSon/release/cni-plugins-linux-amd64-v0.9.0.tgz Rong/pre-rong/rong_static/for_cluster/
# cp RobotSon/release/kube* Rong/pre-rong/rong_static/for_cluster/
```
创建离线docker镜像包并替代原有离线镜像包:     

```
# cd RobotSon/data
# tar czvf docker.tar.gz docker/
# cd ../../
# rm -f Rong/pre-rong/rong_static/for_master0/docker.tar.gz
# mv RobotSon/data/docker.tar.gz Rong/pre-rong/rong_static/for_master0/
```

更改`rong/1_preinstall/roles/preinstall/tasks/main.yml`, 更改为新的静态包.        

替换`rong/3_k8s`目录:     

```
# tar xzvf kubespray-2.15.0.tar.gz
# rm -rf rong/3_k8s/
# mv kubespray-2.15.0/* rong/3_k8s/
```
更改`bootstrap`角色:    

```
# cp ./rong/3_k8s/roles/bootstrap-os/tasks/main.yml ./rong/3_k8s/roles/bootstrap-os/task/main_main.yml
# cp /run/media/dash/aa3eda99-dc11-4c07-a5f1-d00eb0acc850/Rong_V7.0.0/Origin/rong/3_k8s/roles/bootstrap-os/tasks/main_kfz.yml ./rong/3_k8s/roles/bootstrap-os/tasks/
# cp /run/media/dash/aa3eda99-dc11-4c07-a5f1-d00eb0acc850/Rong_V7.0.0/Origin/rong/3_k8s/roles/bootstrap-os/tasks/main.yml ./rong/3_k8s/roles/bootstrap-os/tasks/
```
更改`container-engine/docker`角色，与上差不多的步骤。

更改`rong-vars.yml`里的相关定义:     

```
kubeadm_download_url:
kubelet_download_url:
kubectl_download_url:
helm_download_url:

helm_enabled: true
#helm_version: "v2.16.1"
helm_skip_refresh: true

containerd_version: '1.2.13'
```
