+++
title = "TipsOnJenkinsX"
date = "2018-06-11T10:43:35+08:00"
description = "TipsOnJenkinsX"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Install
Install under archlinux:    

```
# curl -L https://github.com/jenkins-x/jx/releases/download/v1.2.120/jx-linux-amd64.tar.gz | tar xzv 
# sudo mv jx /usr/local/bin
```
upgrading minikube:    

```
# docker-machine-driver-kvm2
```
Now create the cluster via:    

```
# jx create cluster minikube
? cpu (cores) 3
? Select driver: kvm2
WARNING: We cannot yet automate the installation of KVM with KVM2 driver - can you install this manually please?
Please see: https://www.linux-kvm.org/page/Downloads and https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#kvm2-driver
Creating Minikube cluster...

```

### chartmusuem
Install local executable file:    

```
# curl -LO https://s3.amazonaws.com/chartmuseum/release/latest/bin/linux/amd64/chartmuseum
# chmod +x ./chartmuseum
# mv ./chartmuseum /usr/local/bin
```
Initialize with the local filesystem storage:    

```
# sudo mkdir chartstorage
# sudo chartmuseum --debug --port=8988 --storage="local" --storage-local-rootdir="./chartstorage"
```
Now visit your `http://localhost:8988` you could reach the chartmusuem.    

Using chartmusuem:    

```
# helm repo add chartmuseum http://localhost:8988
# helm update
```
Upload chart:    

```
# git clone https://github.com/stakater/chart-mysql.git
# cd chart-mysql
# cd mysql
# helm lint
# helm package .
# curl -L --data-binary "@mysql-1.0.1.tgz" http://localhost:8988/api/charts
```
Now you could see the chart has been uploaded to your own chartmusuem.   
