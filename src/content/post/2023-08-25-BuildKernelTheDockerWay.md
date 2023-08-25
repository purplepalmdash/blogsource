+++
title= "BuildKernelTheDockerWay"
date = "2023-08-25T08:42:23+08:00"
description = "BuildKernelTheDockerWay"
keywords = ["Technology"]
categories = ["Technology"]
+++
使用`rockylinux:9`的容器镜像创建一个容器实例:   

```
sudo docker run -it rockylinux:9 bash
```
在容器实例中，运行以下命令准备内核的编译环境:    

```
sed -e 's|^mirrorlist=|#mirrorlist=|g'     -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g'     -i.bak     /etc/yum.repos.d/rocky-extras.repo     /etc/yum.repos.d/rocky.repo
yum makecache
dnf install -y 'dnf-command(config-manager)'
dnf config-manager --set-enabled crb
yum install -y vim rpm-build python3-devel elfutils-devel  openssl-devel perl-generators pesign yum-utils bc bison bpftool dwarves flex gcc gcc-c++ git-core hmaccalc kmod m4 make net-tools perl-devel gcc-plugin-devel  rpm-build rpmdevtools  dnf-plugins-core ncurses-devel make gcc bc bison flex elfutils-libelf-devel openssl-devel grub2 rpm-build rsync gcc vim yum-utils perl systemd-udev  asciidoc audit-libs-devel binutils-devel clang dwarves fuse-devel gcc-c++ gcc-plugin-devel git-core glibc-static java-devel kabi-dw kernel-rpm-macros libbabeltrace-devel libbpf-devel libcap-devel libcap-ng-devel libmnl-devel libnl3-devel libtraceevent-devel libtracefs-devel lld llvm lvm2 net-tools newt-devel numactl-devel pciutils-devel perl-devel python3-docutils system-sb-certs tpm2-tools xmlto elfutils-devel nss-tools perl-generators pesign python3-devel xz-devel
# download the following packages offlinely 
yum install -y WALinuxAgent-cvm-2.7.0.6-9.el9_2.1.rocky.0.noarch.rpm systemd-boot-unsigned-252-14.el9_2.1.x86_64.rpm
useradd -m mock
```
新建一个终端，在该终端上将运行中且已做上述修改的容器实例提交为容器镜像以便下次使用:     

```
[root@dellnew ~]# docker ps 
CONTAINER ID   IMAGE          COMMAND   CREATED         STATUS         PORTS     NAMES
f7eb549f3d44   rockylinux:9   "bash"    7 minutes ago   Up 7 minutes             wonderful_sinoussi
[root@dellnew ~]# docker commit wonderful_sinoussi buidrockykernel:latest
[root@dellnew ~]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED         SIZE
buidrockykernel   latest    207a4b57059e   5 seconds ago   1.94GB
```

### 2. 使用容器编译内核
使用上节创建的编译镜像编译内核:    

```
[root@text ~]# docker run --name=testrocky -v /root/buildout:/buildout -it buidrockykernel:latest /bin/bash
[root@fa4d8f532c21 /]# cp /buildout/kernel-5.15.113-200.el9.src.rpm /home/mock/
[root@fa4d8f532c21 /]# su - mock
[mock@fa4d8f532c21 ~]$ rpm -Uvh kernel-5.15.113-200.el9.src.rpm 
[mock@fa4d8f532c21 ~]$ cd rpmbuild/SPECS/
[mock@fa4d8f532c21 SPECS]$ time rpmbuild -ba kernel.spec 2>&1 | tee build.log
```
编译出的内核rpm包位于/home/mock/rpmbuild下，可以通过`find /home/mock/rpmbuild | grep rpm$`命令找到。


