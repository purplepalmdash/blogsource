+++
title= "DockerizeUOS"
date = "2023-07-19T08:15:42+08:00"
description = "DockerizeUOS"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install the server via :    

![/images/2023_07_19_08_15_49_457x277.jpg](/images/2023_07_19_08_15_49_457x277.jpg)

关机后，在host机器上，:    

```
apt install -y docker.io guestfish
qemu-img convert -f qcow2 -O raw  uos10G.qcow2 uos10G.img
guestfish -a uos10G.img --ro

Welcome to guestfish, the guest filesystem shell for
editing virtual machine filesystems and disk images.

Type: ‘help’ for help on commands
      ‘man’ to read the manual
      ‘quit’ to quit the shell

><fs> run
><fs> list-filesystems
/dev/sda1: ext4
/dev/sda2: ext4
><fs> mount /dev/sda2 /
><fs> mount /dev/sda1 /boot
><fs> tar-out / - | xz --best >> myuos.xz
><fs> exit
 cat myuos.xz | docker import - uoskkk
root@delli9:~# docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
uoskkk     latest    2fb3905a142f   18 seconds ago   1.68GB
```
run into docker instance via:    

```
root@delli9:~# docker run -it uosctyun:latest /bin/bash
[root@8d94931f1eb0 /]# cat /etc/issue
\S
Kernel \r on an \m
[root@8d94931f1eb0 /]# cat /etc/uos-release 
UOS Server Enterprise-C 20
[root@8d94931f1eb0 /]# yum makecache
```

使用方法, 绿色版安装docker:    

```
[root@uos ~]# tar xzvf docker-24.0.2.tgz 
docker/
docker/docker-proxy
docker/containerd-shim-runc-v2
docker/ctr
docker/docker
docker/docker-init
docker/runc
docker/dockerd
docker/containerd
[root@uos ~]# mv docker/* /usr/bin
[root@uos ~]# dockerd&

```
在另一个终端上启动容器实例:     

```
[root@uos ~]# cat myuos.xz | docker import - uosxj
sha256:e09cca9384977ce87a05d50a50134a8fa44607b19f4586222b835916dddb24a0
[root@uos ~]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
uosxj        latest    e09cca938497   40 seconds ago   1.68GB
[root@uos ~]# docker run --privileged -it uosxj /bin/bash
[root@0bb3783f7e1e /]# yum makecache
[root@0bb3783f7e1e /]# yum install -y qemu-kvm-ev
[root@0bb3783f7e1e /]# /usr/libexec/qemu-kvm  --version
QEMU emulator version 2.12.0 (qemu-kvm-ev-2.12.0-45.uelc20_2.01)
Copyright (c) 2003-2017 Fabrice Bellard and the QEMU Project developers

```
