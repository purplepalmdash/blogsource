+++
date = "2017-05-24T15:28:51+08:00"
keywords = ["Virtualization"]
description = "Dev envs for lab"
title = "DevEnvsForLAB"
categories = ["Virtualization"]

+++
### 目的
LAB验证环境的搭建。    
前提条件： 最小化CentOS7系统迁移（见前一篇文章）.    

### 安装
CentOS 7.3(1611), 最小化安装。    

```
# yum update -y
# yum install -y vim qemu libvirt libvirt-devel ruby-devel gcc qemu-kvm
net-tools virt-manager wget lm_sensors iotop
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# yum install -y nethogs byobu ansible rubygem-ruby-libvirt.x86_64
# wget https://releases.hashicorp.com/vagrant/1.9.1/vagrant_1.9.1_x86_64.rpm
# yum install vagrant_1.9.1_x86_64.rpm
# vagrant plugin install --plugin-version  0.0.37 vagrant-libvirt
# cd ~/.vagrant.d/gems/2.2.5/gems
# ln -s ../extensions ./
# vi /etc/modprobe.d/kvm-nested.conf
options kvm_intel nested=1
```
Disable the selinux, firewalld:    

```
# vim /etc/selinux/config
SELINUX=disabled
# systemctl disable firewalld
```
Now restart the machine, your dev environment is ready now.    

Install desktop(MATE):    

```
$ yum groupinstall "MATE Desktop" -y
$ yum groupinstall "X Window System" -y
# systemctl isolate graphical.target
# systemctl set-default graphical.target
# yum install -y gvim gedit gimp 
# wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo
# sed -i "s/gpgkey=https/gpgkey=http/" /etc/yum.repos.d/virtualbox.repo
# yum install  gcc make patch  dkms qt libgomp
# yum install -y kernel-headers kernel-devel fontforge binutils glibc-headers
glibc-devel VirtualBox-5.1 tigervnc tigervnc-server tigervnc tigervnc-server
# yum install -y wireshark tcpdump iftop  python-epdb.noarch python3-rpdb.noarch python2-rpdb.noarch sysstat libreoffice golang unzip htop wireshark-gnome zsh ddd gdb git subversion 
# yum install epel-release
# rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
# yum install simplescreenrecorder
# yum install -y smplayer
```

Install Docker(stable version):    

```
# curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
# systemctl enable docker
```
Install xfce4 desktop:    

```
# yum groupinstall "Xfce" -y
```

### USB启动问题
CentOS 7的initramfs需要重新编译，以获得usb支持。步骤如下:    

获取Linux Kernel版本:    

```
# ls /lib/modules
3.10.0-514.el7.x86_64
```
在`/boot/`目录下重新生成initramfs:    

```
# mkinitrd --with-usb --preload=ehci-hcd --preload=usb-storage --preload=scsi_mod --preload=sd_mod ./usbinitrd-3.10.0-514.el7.x86_64 3.10.0-514.el7.x86_64
```
更改grub配置项:    

```
# vim /boot/grub2/grub.cfg
	- initrd16 /boot/initramfs-3.10.0-514.el7.x86_64.img
	+ initrd16 /boot/usbinitrd-3.10.0-514.el7.x86_64.img
```
现在重新启动，则可以通过USB启动系统。    

### megaraid sas issue
The server has the megaraid sas 2208:    

```
LSI Logic / Symbios Logic MegaRAID SAS 2208 [Thunderbolt] (rev 05)
```
which could not be recognized via CentOS 7.3, thus we have to install
following packages:    

```
# yum install kmod-redhat-megaraid_sas.x86_64
```
I thinks this could solve the problems.   

Or we could use the rescue kernel for booting the system, which acts the same
as the general kernels

No, install `kmod-redhat-megaraid_sas.x86_64` won't solve the problem, so I
googled and finf following way could solves this problem.    

```
# dracut --add-driver megaraid_sas.ko -f
/boot/initramfs-3.10.0-514.21.1.el7.x86_64.img 3.10.0-514.21.1.el7.x86_64
```
### TBD
ScreeShot Software
