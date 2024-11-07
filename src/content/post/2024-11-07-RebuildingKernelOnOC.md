+++
title= "RebuildingKernelOnOC"
date = "2024-11-07T23:17:19+08:00"
description = "RebuildingKernelOnOC"
keywords = ["Technology"]
categories = ["Technology"]
+++
Steps:    

```
sudo yum update -y
sudo yum install bc bison dwarves flex git ncurses-devel.x86_64 rpm-build rsync wget -y
sudo yum groupinstall "Development Tools" -y
sudo yum install openssl perl -y
yum install -y openssl-dev
```
Get the source code:    

```
     wget https://mirrors.ustc.edu.cn/kernel.org/linux/kernel/v5.x/linux-5.15.tar.xz
tar xJvf linux-5.15.tar.xz
cd linux-5.15
cp -v /boot/config-$(uname -r)* .config
make menuconfig
scripts/config --disable DEBUG_INFO
scripts/config --set-str SYSTEM_TRUSTED_KEYS ""
scripts/config --set-str SYSTEM_REVOCATION_KEYS ""
make menuconfig
```
Using old version of pahole(1.23):    

```
yum remove dwarves
wget https://git.kernel.org/pub/scm/devel/pahole/pahole.git/snapshot/pahole-1.23.tar.gz
tar xzvf pahole-1.23.tar.gz
cd pahole-1.23
cd lib/bpf
wget https://github.com/libbpf/libbpf/archive/refs/tags/v0.6.0.zip
unzip libbpf-0.6.0.zip
mv libbpf-0.6.0/* .
cd ../../
mkdir build
cd build
 cmake -D__LIB=lib -DCMAKE_INSTALL_PREFIX=/usr -DBUILD_SHARED_LIBS=ON ..
make install
cp /usr/lib/libdwarves* /usr/lib64/
[root@localhost build]# which pahole
/usr/local/bin/pahole
[root@localhost build]# pahole --version
v1.23
```
Now rebuild the kernel, enable the option:    

![/images/2024_11_07_23_24_56_894x257.jpg](/images/2024_11_07_23_24_56_894x257.jpg)

```
make -j12 binrpm-pkg LOCALVERSION=-test
```
Get the rpm:    

```
# find /root/rpmbuild/ | grep rpm$
/root/rpmbuild/RPMS/x86_64/kernel-headers-5.15.0_test-1.x86_64.rpm
/root/rpmbuild/RPMS/x86_64/kernel-5.15.0_test-1.x86_64.rpm
/root/rpmbuild/RPMS/x86_64/kernel-headers-5.15.0_test-3.x86_64.rpm
/root/rpmbuild/RPMS/x86_64/kernel-5.15.0_test-3.x86_64.rp
```
Install(remove and re-install):    

```
[root@localhost linux-5.15]# yum remove kernel-5.15.0_test-1.x86_64
[root@localhost linux-5.15]# yum install -y /root/rpmbuild/RPMS/x86_64/kernel-5.15.0_test-3.x86_64.rpm
# reboot
```
![/images/2024_11_07_23_29_34_1082x489.jpg](/images/2024_11_07_23_29_34_1082x489.jpg)

![/images/2024_11_07_23_31_08_1100x610.jpg](/images/2024_11_07_23_31_08_1100x610.jpg)

