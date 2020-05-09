+++
categories = ["Technology"]
date = "2020-05-06T09:09:26+08:00"
description = "WorkingtipsOnkubespray-2.13.0"
keywords = ["Technology"]
title= "WorkingtipsOnkubespray-2.13.0"

+++
### Vagrant machine
vagrant machine is created as `192.168.121.251`, 6-core, 8192 MB Memory, with base images ubuntu18.04.4    

### Steps
Configure the `/etc/apt/sources.lists` for using cn repository, then install pip for python, then install the ansible environment:    

```
# sudo apt-get install -y python-pip
# sudo su
# mkdir -p ~/.pip
# vim ~/.pip/pip.conf
[global]
trusted-host =  mirrors.aliyun.com
index-url = http://mirrors.aliyun.com/pypi/simple
# tar xzvf kubespray-2.13.0.tar.gz
# cd kubespray-2.13.0
# pip install -r requirements.txt
```
Configure the password-less login:    

```
# vim /etc/ssh/sshd_config
PermitRootLogin yes
# systemctl restart sshd
# ssh-keygen
# ssh-copy-id root@192.168.121.251
```
Make sure all of your networking environment could reach out of The fucking GreatFileWall.   


Configure the inventory.ini for deploying:    

```
# vim inventory/sample/inventory.ini
[all]
kubespray ansible_host=192.168.121.251 ip=192.168.121.251

[kube-master]
kubespray

[etcd]
kubespray

[kube-node]
kubespray

[calico-rr]

[k8s-cluster:children]
kube-master
kube-node
calico-rr
# ansible-playbook -i inventory/sample/hosts.ini cluster.yml
```
By now we got all of the offline docker images and allmost all of the debs files, but we have install additional pkgs for our own offline purpose usage:     

```
# apt-get install -y iputils-ping nethogs python-netaddr build-essential bind9 bind9utils nfs-common nfs-kernel-server ntpdate ntp tcpdump iotop unzip wget apt-transport-https socat rpcbind arping fping python-apt ipset ipvsadm pigz nginx docker-registry
# apt-get install -y ./netdata_1.18.1_amd64_bionic.deb
```
Transfer all of the offline debs files and rename it in Rong/ Directory and xz it as `1804debs.tar.xz`.      

Replace(1804debs.tar.xz and `kube*`, and calicoctl/cni-plugin, docker.tar.gz):    

```
# ls
calicoctl                           gpg                    kubectl-v1.17.5-amd64
cni-plugins-linux-amd64-v0.8.5.tgz  kubeadm-v1.17.5-amd64  kubelet-v1.17.5-amd64
# cd ../for_master0/
# ls
1804debs.tar.xz  ansible-playbook_exe  docker-compose     docker.tar.gz
ansible_exe      autoindex.tar.xz      dockerDebs.tar.gz  portable-ansible-v0.4.1-py2.tar.bz2
```
Generate docker registry offline files(On existing cluster master0):    

```
# systemctl stop docker-registry.servica
# cd /var/lib/docker-registry
# mv docker docker.back
# systemctl start docker-registry.servica
# docker push nginx:1.17
# docker push kubernetesui/dashboard-amd64:v2.0.0
# docker push k8s.gcr.io/kube-proxy:v1.17.5
# docker push k8s.gcr.io/kube-apiserver:v1.17.5
# docker push k8s.gcr.io/kube-controller-manager:v1.17.5
# docker push k8s.gcr.io/kube-scheduler:v1.17.5
# docker push k8s.gcr.io/k8s-dns-node-cache:1.15.12
# docker push calico/cni:v3.13.2
# docker push calico/kube-controllers:v3.13.2
# docker push calico/node:v3.13.2
# docker push kubernetesui/metrics-scraper:v1.0.4
# docker push lachlanevenson/k8s-helm:v3.1.2
# docker push k8s.gcr.io/addon-resizer:1.8.8
# docker push coredns/coredns:1.6.5
# docker push k8s.gcr.io/metrics-server-amd64:v0.3.6
# docker push k8s.gcr.io/cluster-proportional-autoscaler-amd64:1.7.1
# docker push quay.io/coreos/etcd:v3.3.12
# docker push k8s.gcr.io/pause:3.1
# systemctl stop docker-registry.servica
# du -hs docker/
484M	docker
# tar czvf docker.tar.gz docker/
```
Kubeadm signature:    

```
# cd /root && wget https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz
# tar xzvf go1.14.2.linux-amd64.tar.gz
# vim /root/.profile
PATH="$PATH:/root/go/bin/"
# source ~/.profile
# go version
go version go1.14.2 linux/amd64
# wget https://github.com/kubernetes/kubernetes/archive/v1.17.5.zip
# unzip v1.17.5.zip
# cd kubernetes-1.17.5/
```
Make code changes for timestamp:    

```
# diff kubernetes-1.17.5/hack/lib/version.sh ../kubernetes-1.17.5/hack/lib/version.sh 
47c47
<     KUBE_GIT_TREE_STATE="archive"
---
>     KUBE_GIT_TREE_STATE="clean"
64c64
<         KUBE_GIT_TREE_STATE="dirty"
---
>         KUBE_GIT_TREE_STATE="clean"
# diff kubernetes-1.17.5/cmd/kubeadm/app/constants/constants.go ../kubernetes-1.17.5/cmd/kubeadm/app/constants/constants.go
47c47
<       CertificateValidity = time.Hour * 24 * 365
---
>       CertificateValidity = time.Hour * 24 * 365 * 100
# diff kubernetes-1.17.5/vendor/k8s.io/client-go/util/cert/cert.go ../kubernetes-1.17.5/vendor/k8s.io/client-go/util/cert/cert.go
66c66
<               NotAfter:              now.Add(duration365d * 10).UTC(),
---
>               NotAfter:              now.Add(duration365d * 100).UTC(),
96c96
<       maxAge := time.Hour * 24 * 365          // one year self-signed certs
---
>       maxAge := time.Hour * 24 * 365 * 100         // one year self-signed certs
110c110
<               maxAge = 100 * time.Hour * 24 * 365 // 100 years fixtures
---
>               maxAge = 100 * time.Hour * 24 * 365  // 100 years fixtures
124c124
<               NotAfter:  validFrom.Add(maxAge),
---
>               NotAfter:  validFrom.Add(maxAge * 100),
152c152
<               NotAfter:  validFrom.Add(maxAge),
---
>               NotAfter:  validFrom.Add(maxAge * 100),
```
Make kubeadm binary files:    

```
# make all WHAT=cmd/kubeadm
# cd _output/bin
# ls
conversion-gen  deepcopy-gen  defaulter-gen  go2make  go-bindata  kubeadm  openapi-gen
# ./kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.5", GitCommit:"e0fccafd69541e3750d460ba0f9743b90336f24f", GitTreeState:"clean", BuildDate:"2020-05-06T03:23:16Z", GoVe
rsion:"go1.14.2", Compiler:"gc", Platform:"linux/amd64"}
```
Code changes mainly in `1_preinstall` and `3_k8s`:    

```
Show diffs., TBD
```
After hanges almost everything will be acts as in old versions.   

### kubespray changes
In new version we have to comment the:     

```
# roles/kubespray-defaults/tasks/main.yaml
# do not run gather facts when bootstrap-os in roles
#- name: set fallback_ips
#  include_tasks: fallback_ips.yml
#  when:
#    - "'bootstrap-os' not in ansible_play_role_names"
#    - fallback_ips is not defined
#  tags:
#    - always
#
#- name: set no_proxy
#  include_tasks: no_proxy.yml
#  when:
#    - "'bootstrap-os' not in ansible_play_role_names"
#    - http_proxy is defined or https_proxy is defined
#    - no_proxy is not defined
#  tags:
#    - always

```
And also change the `container-engine`'s docker roles, thus we won't restart docker to keep the graphical installation on-going:     

```
# cat container-engine/docker/handlers/main.yml 
---
- name: restart docker
  command: echo "HelloWorld"

#  command: /bin/true
#  notify:
#    - Docker | reload systemd
#    - Docker | reload docker.socket
#    - Docker | reload docker
#    - Docker | wait for docker
```
