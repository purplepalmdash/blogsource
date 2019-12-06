+++
title = "Buildingterraform-provider-libvirt"
date = "2019-12-06T15:14:13+08:00"
description = "Buildingterraform-provider-libvirt"
keywords = ["Linux"]
categories = ["Linux"]
+++
Build terraform-provider-libvirt for debian9.0.    

Steps:    

Get system info:    

```
root@debian:~# cat /etc/issue
Debian GNU/Linux 9 \n \l

root@debian:~# cat /etc/debian_version 
9.0

```
wget the terraform and mv it to /usr/bin, then start building the plugin:    

```
# vim /etc/apt/sources.list
deb http://mirrors.163.com/debian/ stretch main non-free contrib
deb http://mirrors.163.com/debian/ stretch-updates main non-free contrib
deb http://mirrors.163.com/debian/ stretch-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ stretch/updates main non-free contrib
# apt-get update -y
# apt-get install libvirt-dev git build-essential golang=2:1.11~1~bpo9+1 golang=2:1.11~1~bpo9+1 golang-doc=2:1.11~1~bpo9+1 golang-go=2:1.11~1~bpo9+1 golang-src=2:1.11~1~bpo9+1
# mkdir /root/go
# vim /root/.bashrc
export GOPATH=/root/go
export PATH=$PATH:$GOPATH/bin
# export CGO_ENABLED="1"
# mkdir -p $GOPATH/src/github.com/dmacvicar; cd $GOPATH/src/github.com/dmacvicar
# git clone https://github.com/dmacvicar/terraform-provider-libvirt.git
# cd $GOPATH/src/github.com/dmacvicar/terraform-provider-libvirt
# make install
```
After building, go to `/root/go/bin` and examine the built plugin:    

```
root@debian:~/go/bin# ./terraform-provider-libvirt --version
./terraform-provider-libvirt e9ff32f1ec5825dcf05481cb7ef6a3b645696a4f-dirty
Compiled against library: libvirt 3.0.0
Using library: libvirt 3.0.0
```
Now you got plugin compiled and ready to use on debian 9.0    
