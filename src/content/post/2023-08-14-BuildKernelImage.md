+++
title= "BuildKernelImage"
date = "2023-08-14T08:20:14+08:00"
description = "BuildKernelImage"
keywords = ["Technology"]
categories = ["Technology"]
+++
Create a docker instance:     

```
sudo docker run -it rockylinux:9 bash
```
In docker, run:    

```
sed -e 's|^mirrorlist=|#mirrorlist=|g'     -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g'     -i.bak     /etc/yum.repos.d/rocky-extras.repo     /etc/yum.repos.d/rocky.repo
yum makecache
dnf install -y 'dnf-command(config-manager)'
dnf config-manager --set-enabled crb
yum install -y vim rpm-build python3-devel elfutils-devel  openssl-devel perl-generators pesign yum-utils bc bison bpftool dwarves flex gcc gcc-c++ git-core hmaccalc kmod m4 make net-tools perl-devel gcc-plugin-devel  rpm-build rpmdevtools  dnf-plugins-core ncurses-devel make gcc bc bison flex elfutils-libelf-devel openssl-devel grub2 rpm-build rsync gcc vim yum-utils perl systemd-udev  asciidoc audit-libs-devel binutils-devel clang dwarves fuse-devel gcc-c++ gcc-plugin-devel git-core glibc-static java-devel kabi-dw kernel-rpm-macros libbabeltrace-devel libbpf-devel libcap-devel libcap-ng-devel libmnl-devel libnl3-devel libtraceevent-devel libtracefs-devel lld llvm lvm2 net-tools newt-devel numactl-devel pciutils-devel perl-devel python3-docutils system-sb-certs tpm2-tools xmlto elfutils-devel nss-tools perl-generators pesign python3-devel xz-devel
# download the following packages offlinely 
yum install -y WALinuxAgent-cvm-2.7.0.6-9.el9_2.1.rocky.0.noarch.rpm systemd-boot-unsigned-252-14.el9_2.1.x86_64.rpm
useradd -m mock
```
Then in host machine, do :     

```
[root@dellnew ~]# docker ps 
CONTAINER ID   IMAGE          COMMAND   CREATED         STATUS         PORTS     NAMES
f7eb549f3d44   rockylinux:9   "bash"    7 minutes ago   Up 7 minutes             wonderful_sinoussi
[root@dellnew ~]# docker commit wonderful_sinoussi buidrockykernel:latest
[root@dellnew ~]# docker images
REPOSITORY        TAG       IMAGE ID       CREATED         SIZE
buidrockykernel   latest    207a4b57059e   5 seconds ago   1.94GB
```
Next time using this latest commited kernel you could directly build kernel src.   

### test images
Build via following command:    

```
[root@text ~]# docker run --name=testrocky -v /root/buildout:/buildout -it buidrockykernel:latest /bin/bash
[root@fa4d8f532c21 /]# cp /buildout/kernel-5.15.113-200.el9.src.rpm /home/mock/
[root@fa4d8f532c21 /]# su - mock
[mock@fa4d8f532c21 ~]$ rpm -Uvh kernel-5.15.113-200.el9.src.rpm 
[mock@fa4d8f532c21 ~]$ cd rpmbuild/SPECS/
[mock@fa4d8f532c21 SPECS]$ time rpmbuild -ba kernel.spec 2>&1 | tee build.log
```

