+++
title= "BuildKernelOnRockyLinux92"
date = "2023-08-10T10:56:42+08:00"
description = "BuildKernelOnRockyLinux92"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Try Steps
Replace the repository for speedup:     

```
sed -e 's|^mirrorlist=|#mirrorlist=|g'     -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g'     -i.bak     /etc/yum.repos.d/rocky-extras.repo     /etc/yum.repos.d/rocky.repo
yum makecache
```
Install docker:    

```
dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
# replace all of the `download.docker.com` into `https://mirrors.ustc.edu.cn/docker-ce`
yum makecache
yum install -y docker-ce
systemctl enable docker
systemctl start docker
```
Pull rockylinux9 and prepare building environment via:    

```
docker pull rockylinux:9
docker run -it -v /root/source:/source rockylinux:9 /bin/bash
sed -e 's|^mirrorlist=|#mirrorlist=|g'     -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g'     -i.bak     /etc/yum.repos.d/rocky-extras.repo     /etc/yum.repos.d/rocky.repo
yum makecache
yum install -y rpm-build rpmdevtools  dnf-plugins-core ncurses-devel make gcc bc bison flex elfutils-libelf-devel openssl-devel grub2 rpm-build rsync gcc vim yum-utils
rpm -ivh kernel-5.15.113_lts2021_iotg-1.src.rpm
useradd -m dash
su dash
$ cd 
$ rpmdev-setuptree
$ ls ~
rpmbuild

```

### RockyLinux Way
Create docker instance:     

```
docker run -it -v /root/source:/source rockylinux:9 /bin/bash
```
In docker, prepare for building kernel:   

```
sed -e 's|^mirrorlist=|#mirrorlist=|g'     -e 's|^#baseurl=http://dl.rockylinux.org/$contentdir|baseurl=https://mirrors.ustc.edu.cn/rocky|g'     -i.bak     /etc/yum.repos.d/rocky-extras.repo     /etc/yum.repos.d/rocky.repo
yum makecache
yum install -y rpm-build rpmdevtools  dnf-plugins-core ncurses-devel make gcc bc bison flex elfutils-libelf-devel openssl-devel grub2 rpm-build rsync gcc vim yum-utils perl
yum --enablerepo=crb install -y systemd-udev  asciidoc audit-libs-devel binutils-devel clang dwarves fuse-devel gcc-c++ gcc-plugin-devel git-core glibc-static java-devel kabi-dw kernel-rpm-macros libbabeltrace-devel libbpf-devel libcap-devel libcap-ng-devel libmnl-devel libnl3-devel libtraceevent-devel libtracefs-devel lld llvm lvm2 net-tools newt-devel numactl-devel pciutils-devel perl-devel python3-docutils system-sb-certs tpm2-tools xmlto elfutils-devel nss-tools perl-generators pesign python3-devel xz-devel

yum install /source/WALinuxAgent-cvm-2.7.0.6-9.el9_2.1.rocky.0.noarch.rpm /source/systemd-boot-unsigned-252-14.el9_2.1.x86_64.rpm
useradd -m mock
su - mock
```
Download the kernel src rpm:     

```
$ yumdownloader --source kernel
$ ls -l -h
total 139M
-rw-r--r--. 1 mock mock 139M Aug 10 06:30 kernel-5.14.0-284.25.1.el9_2.src.rpm
$ rpm -Uvh kernel-5.14.0-284.25.1.el9_2.src.rpm 
[mock@c51507a516d4 ~]$ cd ./rpmbuild/SPECS/
[mock@c51507a516d4 SPECS]$ ls
kernel.spec
[mock@c51507a516d4 SPECS]$ time rpmbuild -ba kernel.spec 

```

### intel way
using the `kernel-5.15.113-1.src.rpm` package we got following issue:    

```
Processing files: kernel-headers-5.15.113-1.x86_64
Provides: kernel-headers = 5.15.113 kernel-headers = 5.15.113-1 kernel-headers(x86-64) = 5.15.113-1
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
Obsoletes: kernel-headers
Processing files: kernel-devel-5.15.113-1.x86_64
warning: absolute symlink: /lib/modules/5.15.113/build -> /usr/src/kernels/5.15.113
warning: absolute symlink: /lib/modules/5.15.113/source -> /usr/src/kernels/5.15.113
Provides: kernel-devel = 5.15.113-1 kernel-devel(x86-64) = 5.15.113-1
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
Processing files: kernel-debugsource-5.15.113-1.x86_64
error: Could not open %files file /home/mock/rpmbuild/BUILD/kernel-5.15.113/debugsourcefiles.list: No such file or directory


RPM build errors:
    line 20: It's not recommended to have unversioned Obsoletes: Obsoletes: kernel-headers
    absolute symlink: /lib/modules/5.15.113/build -> /usr/src/kernels/5.15.113
    absolute symlink: /lib/modules/5.15.113/source -> /usr/src/kernels/5.15.113
    Could not open %files file /home/mock/rpmbuild/BUILD/kernel-5.15.113/debugsourcefiles.list: No such file or directory

```
### fedora 5.15 lts
Refers to `https://copr.fedorainfracloud.org/coprs/kwizart/kernel-longterm-5.15/`.    

```
yum install -y bpftool
dnf copr enable kwizart/kernel-longterm-5.15
yumdownloader --source kernel-longterm
su - mock
rpm -Uvh kernel-longterm-5.15.124-200.el9.src.rpm
cd /home/mock/rpmbuild
time rpmbuild -ba kernel.spec 
```
build result:   

```
$ find . | grep rpm$
./RPMS/x86_64/kernel-longterm-modules-internal-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-debug-devel-matched-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-debug-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-devel-matched-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-debug-modules-extra-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-modules-extra-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-debug-modules-internal-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-debug-debuginfo-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-debug-devel-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-debuginfo-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-devel-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-debug-modules-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-debuginfo-common-x86_64-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-modules-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-debug-core-5.15.124-200.el9.x86_64.rpm
./RPMS/x86_64/kernel-longterm-core-5.15.124-200.el9.x86_64.rpm
./SRPMS/kernel-longterm-5.15.124-200.el9.src.rpm
```

