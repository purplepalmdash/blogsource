+++
title = "WorkingTipsOnFfDL"
date = "2019-05-21T12:02:59+08:00"
description = "WorkingTipsOnFfDL"
keywords = ["Linux"]
categories = ["Technology"]
+++
### StartPoint
Working directory:    

```
# /home/xxxx/Code/vagrant/ai_k8s/RONG/package/files/Rong
# vagrant status
Current machine states:

outnode-1                 running (libvirt)
```

A running k8s cluster:    

```
[root@outnode-1 ~]# cat /etc/redhat-release 
Red Hat Enterprise Linux Server release 7.4 (Maipo)
[root@outnode-1 ~]# kubectl get nodes
NAME        STATUS   ROLES    AGE   VERSION
outnode-1   Ready    master   32m   v1.14.1
```
Configure repository:    

```
# mount /dev/sr0 /mnt
# cd /etc/yum.repos.d
# mv *.repo /root/
# vim cdrom.repo
[local]
name=local
baseurl=file:///mnt
enabled=1
gpgcheck=0
# yum makecache
# yum install -y vim git nfs-utils rpcbind
```
Configure nfs server:    

```
# mkdir -p /opt/nfs
# vim /etc/exports
/opt/nfs  *(rw,async,no_root_squash,no_subtree_check)
# service rpcbind start
# service nfs start
# systemctl enable nfs-server
# systemctl start nfs-server
# systemctl enable nfs.service
# systemctl enable rpcbind
```

Configure helm via:    

```
# helm repo add stable https://kubernetes-charts.storage.googleapis.com
# helm install stable/nfs-client-provisioner --set nfs.server=10.142.108.191 --set nfs.path=/opt/nfs
# kubectl get sc
nfs-client   cluster.local/righteous-condor-nfs-client-provisioner   7m8s
# kubectl edit sc nfs-client
kind: StorageClass
metadata:
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
```

### Install
Clone the source code from github:    

```
# git clone https://github.com/IBM/FfDL.git
# 

```

TOO many errors here. To be continue.  
