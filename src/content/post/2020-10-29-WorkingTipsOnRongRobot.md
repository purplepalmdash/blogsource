+++
title= "WorkingTipsOnRongRobot"
date = "2020-10-29T12:37:57+08:00"
description = "WorkingTipsOnRongRobot"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Building
In Azure Devops, Create new project:   

![/images/2020_10_29_12_39_14_651x545.jpg](/images/2020_10_29_12_39_14_651x545.jpg)

Create pipeline:   

![/images/2020_10_29_12_39_37_449x459.jpg](/images/2020_10_29_12_39_37_449x459.jpg)

Select code for `GitHub`:    

![/images/2020_10_29_12_40_07_665x532.jpg](/images/2020_10_29_12_40_07_665x532.jpg)

Authorized AzurePipeLines:    

![/images/2020_10_29_12_40_42_519x188.jpg](/images/2020_10_29_12_40_42_519x188.jpg)

Select Repository:    

![/images/2020_10_29_12_41_19_706x269.jpg](/images/2020_10_29_12_41_19_706x269.jpg)

Click `Run`:  

![/images/2020_10_29_12_41_49_894x394.jpg](/images/2020_10_29_12_41_49_894x394.jpg)

View Status:    

![/images/2020_10_29_12_43_19_859x544.jpg](/images/2020_10_29_12_43_19_859x544.jpg)

Running Status:  

![/images/2020_10_29_12_43_39_835x460.jpg](/images/2020_10_29_12_43_39_835x460.jpg)

Check Result:  

![/images/2020_10_29_13_23_16_752x254.jpg](/images/2020_10_29_13_23_16_752x254.jpg)

![/images/2020_10_29_13_23_44_839x546.jpg](/images/2020_10_29_13_23_44_839x546.jpg)


Check Artifacts:    

![/images/2020_10_29_13_24_05_754x257.jpg](/images/2020_10_29_13_24_05_754x257.jpg)

Download Artifacts:   

![/images/2020_10_29_13_24_25_850x299.jpg](/images/2020_10_29_13_24_25_850x299.jpg)


### Patching
#### Static file Patching

After download:    

```
 $ ls *
RobotSon.tar.gz

data:
docker 

release:
calicoctl  cni-plugins-linux-amd64-v0.8.7.tgz  kubeadm-v1.19.3-amd64  kubectl-v1.19.3-amd64  kubelet-v1.19.3-amd64
```
zip `docker.tar.gz`(place in `pre-rong/rong_static/for_master0/docker.tar.gz`)

```
$ cd data
$ tar czf docker.tar.gz docker/
```
Copy `releases` folder to folder(`pre-rong/rong_static/for_cluster/`)

```
$  ls pre-rong/rong_static/for_cluster/
calicoctl  cni-plugins-linux-amd64-v0.8.7.tgz  docker  gpg  kubeadm-v1.18.8-amd64  kubectl-v1.18.8-amd64  kubelet-v1.18.8-amd64  netdata-v1.22.1.gz.run
```
#### Code Patching
下载patch文件: 

```
# git clone https://github.com/kubernetes-sigs/kubespray.git
# cd kubespray
# git checkout tags/v2.xx.0 -b xxxx
# git apply --check ../patch 
检查是否有错
v1.19(master)需要exclude以下两个文件
# git apply  /root/patch --exclude=roles/kubernetes-apps/helm/templates/tiller-clusterrolebinding.yml.j2 --exclude=roles/remove-node/remove-etcd-node/tasks/main.yml
```
部署框架内少量修改

`rong-vars.yml`:   

![/images/2020_10_29_14_12_23_871x347.jpg](/images/2020_10_29_14_12_23_871x347.jpg)

![/images/2020_10_29_14_12_36_425x97.jpg](/images/2020_10_29_14_12_36_425x97.jpg)


`rong/1_preinstall/role/preinstall/task/main.yml`:   

![/images/2020_10_29_14_11_39_875x477.jpg](/images/2020_10_29_14_11_39_875x477.jpg)

