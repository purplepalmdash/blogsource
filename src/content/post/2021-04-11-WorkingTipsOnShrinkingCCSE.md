+++
title= "WorkingTipsOnShrinkingCCSE"
date = "2021-04-11T16:51:48+08:00"
description = "WorkingTipsOnShrinkingCCSE"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Repository Shrinking
Create a new vm via following command:    

```
# cd /var/lib/libvirt/qemu/save
### Following is for creating a new vm for saving rpms
# virsh dumpxml node1>example.xml
# vim example.xml 
# qemu-img create -f qcow2 -b ccsebaseimage.qcow2 saverpms.qcow2 
Formatting 'saverpms.qcow2', fmt=qcow2 size=536870912000 backing_file=ccsebaseimage.qcow2 cluster_size=65536 lazy_refcounts=off refcount_bits=16
# virsh define example.xml 
Domain nodetmp defined from example.xml
# virsh start nodetmp
Domain nodetmp started
# virsh net-dhcp-leases default
### Getting the ip address for nodetmp(10.17.18.199)
# scp ./ccse-offline-files.tar.gz root@10.17.18.199:/home/
# ssh root@10.17.18.199
```
Following is on `10.17.18.199`:   

```
[root@first ~]# cd /home/
[root@first home]# tar xzvf ccse-offline-files.tar.gz
# vi /etc/yum.conf
keepcache=1
### Add a new vm disk (vdb)
# fdisk /dev/vdb
# mkfs.ext4 /dev/vdb1
# mkdir /dcos
# mount /dev/vdb1 /dcos
# vi /etc/fstab 
/dev/vdb1 /dcos                   ext4     defaults        0 0
# mount -a
# exit
```
Bug-fix(lsof):    

```
# scp ./Packages/lsof-4.87-6.el7.x86_64.rpm root@10.17.18.199:/root/
```

Backup the vm disks on host machine:    

```
# virsh destroy nodetmp
# mv saverpms.qcow2 saverpms1.qcow2
# qemu-img create -f qcow2 -b saverpms1.qcow2 saverpms.qcow2
# virsh start nodetmp
# ssh root@10.17.18.199
```
Re-login, and run:    

```
#  rpm -ivh /root/lsof-4.87-6.el7.x86_64.rpm 
# cd /home/ccse-xxxxxxxx
# vi config/config.yaml
common:
  # 控制台和/或Harbor所在的主机IP
  host: 10.17.18.199
# vim ./files/offline-repo/ccse-centos7-base.repo
	#[ccse-centos7-base]
	#name=ccse-offline-repo
	#baseurl=file://{centos7_base_repo_dir}
	#enabled=1
	#gpgcheck=0
	[ccse-centos7-base]
	name=Centos local yum repo for k8s
	baseurl=http://10.17.18.2:8200/repo/x86_64/centos7-base
	gpgcheck=0
	enabled=1
	proxy=_none_
# ./deploy.sh install all 2>&1 | sudo tee install-log_`date "+%Y%m%d%H%M"`
```
Notice, `10.17.18.2` is for existing ccse console.  
After deployment, the cached rpms is listed as:    

```
# find /var/cache | grep rpm$
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/audit-2.8.5-4.el7.x86_64.rpm
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/audit-libs-2.8.5-4.el7.x86_64.rpm
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/audit-libs-python-2.8.5-4.el7.x86_64.rpm
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/checkpolicy-2.5-8.el7.x86_64.rpm
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/libsemanage-python-2.5-14.el7.x86_64.rpm
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/policycoreutils-2.5-34.el7.x86_64.rpm
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/policycoreutils-python-2.5-34.el7.x86_64.rpm
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/python-IPy-0.75-6.el7.noarch.rpm
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/setools-libs-3.3.8-4.el7.x86_64.rpm
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/libcgroup-0.41-21.el7.x86_64.rpm
/var/cache/yum/x86_64/7/ccse-centos7-base/packages/unzip-6.0-21.el7.x86_64.rpm
```
Now enable the visit for ccse console(web ui):    

```
# systemctl stop firewalld
# systemctl disable firewalld
# setenforce 0
# vi /etc/selinux/config 
SELINUX=disabled
```
ccse webui:   

![/images/2021_04_12_10_37_55_515x298.jpg](/images/2021_04_12_10_37_55_515x298.jpg)

Create a new vm and added it on ccse webui, in newly added vm do following command:    

```
# vi /etc/yum.conf 
keepcached
# systemctl stop firewalld
# systemctl disable firewalld
# setenforce 0
# vi /etc/selinux/config 
SELINUX=disabled
```
Create a new cluster, and fetch the new vm's rpm cache:   

```
[root@first cache]# find . | grep rpm$
./yum/x86_64/7/ccse-centos7-base/packages/audit-2.8.5-4.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/audit-libs-2.8.5-4.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/checkpolicy-2.5-8.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/audit-libs-python-2.8.5-4.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/libsemanage-python-2.5-14.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/libcgroup-0.41-21.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/policycoreutils-2.5-34.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/python-IPy-0.75-6.el7.noarch.rpm
./yum/x86_64/7/ccse-centos7-base/packages/setools-libs-3.3.8-4.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/policycoreutils-python-2.5-34.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/conntrack-tools-1.4.4-7.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/libnetfilter_cttimeout-1.0.0-7.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/libnetfilter_queue-1.0.2-2.el7_2.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/socat-1.7.3.2-2.el7.x86_64.rpm
./yum/x86_64/7/ccse-centos7-base/packages/libnetfilter_cthelper-1.0.0-11.el7.x86_64.rpm
./yum/x86_64/7/ccse-k8s/packages/container-selinux-2.119.1-1.c57a6f9.el7.noarch.rpm
./yum/x86_64/7/ccse-k8s/packages/docker-ce-18.09.9-3.el7.x86_64.rpm
./yum/x86_64/7/ccse-k8s/packages/containerd.io-1.2.13-3.2.el7.x86_64.rpm
./yum/x86_64/7/ccse-k8s/packages/docker-ce-cli-18.09.9-3.el7.x86_64.rpm
./yum/x86_64/7/ccse-k8s/packages/3f1db71d0bb6d72bc956d788ffee737714e5717c629b26355a2dcf1dba4ad231-kubelet-1.17.3-0.x86_64.rpm
./yum/x86_64/7/ccse-k8s/packages/548a0dcd865c16a50980420ddfa5fbccb8b59621179798e6dc905c9bf8af3b34-kubernetes-cni-0.7.5-0.x86_64.rpm
./yum/x86_64/7/ccse-k8s/packages/35625b6ab1da6c58ce4946742181c0dcf9ac9b6c2b5bea2c13eed4876024c342-kubectl-1.17.3-0.x86_64.rpm
```



### harbor shrinking
Save the harbor images:    

```
[root@first ~]# docker save -o harbor.tar goharbor/chartmuseum-photon:v0.9.0-v1.8.6 goharbor/harbor-migrator:v1.8.6 goharbor/redis-photon:v1.8.6 goharbor/clair-photon:v2.1.0-v1.8.6 goharbor/notary-server-photon:v0.6.1-v1.8.6 goharbor/notary-signer-photon:v0.6.1-v1.8.6 goharbor/harbor-registryctl:v1.8.6 goharbor/registry-photon:v2.7.1-patch-2819-v1.8.6 goharbor/nginx-photon:v1.8.6 goharbor/harbor-log:v1.8.6 goharbor/harbor-jobservice:v1.8.6 goharbor/harbor-core:v1.8.6 goharbor/harbor-portal:v1.8.6 goharbor/harbor-db:v1.8.6 goharbor/prepare:v1.8.6
[root@first ~]# ls -l -h harbor.tar 
-rw-------. 1 root root 1.5G Apr 11 23:31 harbor.tar
[root@first ~]# cp harbor.tar harbor.tar.back
[root@first ~]# xz -T4 harbor.tar
[root@first ~]# ls -l -h harbor.tar.*
-rw-------. 1 root root 1.5G Apr 11 23:31 harbor.tar.back
-rw-------. 1 root root 428M Apr 11 23:31 harbor.tar.xz
```


### rpm
rpms combine:   

```
[root@first rpms]# ls -l -h | wc -l
12
##### After transferring from working node
#########################################
[root@first rpms]# cp /tmp/rpms/* .
cp: overwrite ‘./audit-2.8.5-4.el7.x86_64.rpm’? y
cp: overwrite ‘./audit-libs-2.8.5-4.el7.x86_64.rpm’? y
cp: overwrite ‘./audit-libs-python-2.8.5-4.el7.x86_64.rpm’? y
cp: overwrite ‘./checkpolicy-2.5-8.el7.x86_64.rpm’? y
cp: overwrite ‘./libcgroup-0.41-21.el7.x86_64.rpm’? y
cp: overwrite ‘./libsemanage-python-2.5-14.el7.x86_64.rpm’? y
cp: overwrite ‘./policycoreutils-2.5-34.el7.x86_64.rpm’? y
cp: overwrite ‘./policycoreutils-python-2.5-34.el7.x86_64.rpm’? y
cp: overwrite ‘./python-IPy-0.75-6.el7.noarch.rpm’? y
cp: overwrite ‘./setools-libs-3.3.8-4.el7.x86_64.rpm’? y
[root@first rpms]# ls -l -h | wc -l
17
```

