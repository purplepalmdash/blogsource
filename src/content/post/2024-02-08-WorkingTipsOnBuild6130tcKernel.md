+++
title= "WorkingTipsOnBuild6130tcKernel"
date = "2024-02-08T11:22:16+08:00"
description = "WorkingTipsOnBuild6130tcKernel"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Preparation
Definition for the vagrant build machine:     

```
$ cat Vagrantfile 
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-22.04"
  config.disksize.size = '180GB'

config.vm.provider "virtualbox" do |v|
  v.memory = 65535
  v.cpus = 12
end

end
$ pwd
/home/dash/Code/vagrant/buildUbuntuKernel610tc

```
Extend the partition after login to the vm:    

```
lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv 
resize2fs /dev/ubuntu-vg/ubuntu-lv
```
update the system:    

```
apt update -y && apt upgrade -y
```

Install necessary packages for building kernel:    
```
sudo apt install libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf llvm
sudo apt build-dep linux linux-image-unsigned-6.1.0-1029-oem
```
### Source Code
enable all of the items of `deb-src` in `/etc/apt/sources.list`, then `sudo apt update -y`.    

```
mkdir -p ~/Code/Code6101029
cd ~/Code/Code6101029
apt source linux-image-unsigned-6.1.0-1029-oem
```
Build the packages:    

```
cd /home/vagrant/Code/Code6101029/linux-oem-6.1-6.1.0
fakeroot debian/rules clean
fakeroot debian/rules binary
```
Check the build result:    

```
$ ls
linux-buildinfo-6.1.0-1033-oem_6.1.0-1033.33_amd64.deb
linux-headers-6.1.0-1033-oem_6.1.0-1033.33_amd64.deb
linux-image-unsigned-6.1.0-1033-oem_6.1.0-1033.33_amd64.deb
linux-modules-6.1.0-1033-oem_6.1.0-1033.33_amd64.deb
linux-modules-ipu6-6.1.0-1033-oem_6.1.0-1033.33_amd64.deb
linux-modules-ivsc-6.1.0-1033-oem_6.1.0-1033.33_amd64.deb
linux-modules-iwlwifi-6.1.0-1033-oem_6.1.0-1033.33_amd64.deb
linux-oem-6.1-headers-6.1.0-1033_6.1.0-1033.33_all.deb
linux-oem-6.1-tools-6.1.0-1033_6.1.0-1033.33_amd64.deb
linux-oem-6.1-tools-host_6.1.0-1033.33_all.deb
linux-tools-6.1.0-1033-oem_6.1.0-1033.33_amd64.deb
```
### Using docker for building
Initialize a docker instance for building:     

```
sudo docker run -it -v /media/sdc/Code/buildkernel:/buildkernel ubuntu:22.04 /bin/bash
```

enter the docker and run:    

```
apt update -y
apt install -y vim
vim /etc/apt/sources.list
apt update -y
apt install libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf llvm debhelper rsync python3-docutils bc libcap-dev git build-essential  asciidoc cpio libjvmti-oprofile0 linux-tools-common default-jdk binutils-dev libbtf1 libdwarf-dev dwarf* pahole libdwarf1 libdwarf++0
# get the dependencies via "dpkg-buildpackage -b"
apt install -y makedumpfile libnewt-dev libdw-dev pkg-config libunwind8-dev liblzma-dev libaudit-dev uuid-dev libnuma-dev zstd fig2dev sharutils python3-dev python3-sphinx python3-sphinx-rtd-theme imagemagick graphviz dvipng fonts-noto-cjk latexmk librsvg2-bin
mkdir -p  /buildkernel/oem && cd /buildkernel/oem
apt source linux-image-unsigned-6.1.0-1029-oem
cd linux-oem-6.1
cd linux-oem-6.1-6.1.0/
time sh -c 'fakeroot debian/rules clean && fakeroot debian/rules binary'
```
build time:    

```
real	32m18.795s
user	309m36.938s
sys	43m52.711s
```
### Customize kernel config file
Fetch the config file:    

```
scp remote_config_files /root/config.common.ubuntu
cp /root/config.common.ubuntu linux-oem-6.1-6.1.0/
```

Edit the files:   

```
# vim debian/rules.d/2-binary-arch.mk
......
	if [ -e $(commonconfdir)/config.common.ubuntu ]; then \
		//cat $(commonconfdir)/config.common.ubuntu $(archconfdir)/config.common.$(arch) $(archconfdir)/config.flavour.$(target_flavour) > $(builddir)/build-$*/.config; \
		cat /root/config.common.ubuntu > $(builddir)/build-$*/.config; \
	else \
......
```
patch :    

```
$ patch -p1 < xxxx.patch
```
Then building.  
