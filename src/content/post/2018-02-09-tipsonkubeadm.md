+++
title = "tipsonkubeadm"
date = "2018-02-09T09:56:46+08:00"
description = "tipsonkubeadm"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Repo
Use kismatic's offline repository for deploying, CentOS 7.4.1708:    

```
[base]
name=Base
baseurl=http://10.15.205.2/base
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[updates]
name=updates
baseurl=http://10.15.205.2/updates
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[docker]
name=docker
baseurl=http://10.15.205.2/docker
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[kubernetes]
name=kubernetes
baseurl=http://10.15.205.2/kubernetes
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[gluster]
name=gluster
baseurl=http://10.15.205.2/gluster
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```
Then install docker-ce via:    

```
# yum install -y --setopt=obsoletes=0  docker-ce-17.03.0.ce-1.el7.centos
# systemctl enable docker && systemctl start docker
```


