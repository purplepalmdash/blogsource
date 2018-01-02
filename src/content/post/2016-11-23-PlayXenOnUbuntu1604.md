+++
categories = ["Virtualization"]
date = "2016-11-23T17:29:07+08:00"
description = "Play Xen On Ubuntu16.04"
keywords = ["Virtualization"]
title = "PlayXenOnUbuntu1604"

+++
### 硬件环境
AMD E-350， 8G内存，320G 硬盘，绝对垃圾配置。     

### 软件环境
Ubuntu16.04, LVM磁盘配置:     

```
$ sudo lvdisplay
.....
.....
......
```

安装完毕后，安装xen hypervisor:     

```
$ sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get dist-upgrade
-y
$ sudo apt-get install -y xen-hypervisor-amd64
$ sudo apt-get install -y virtinst virt-manager
```

更改grub配置，默认使用xen hypervisor内核, 并指定dom0最大可用内存:    

```
$ sudo vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="Xen 4.1-amd64"
GRUB_CMDLINE_XEN="dom0_mem=1024M,max:1024M dom0_max_vcpus=2"
$ sudo update-grub
$ sudo reboot
```
检查:     

```
$ sudo xl list
Name                                        ID   Mem VCPUs	State
Time(s)
Domain-0                                     0  1024     2     r-----
28.7
```
配置一个xenbr0的网桥:    

```
$ sudo vim /etc/network/interface
# The primary network interface
auto enp4s0
iface enp4s0 inet manual

auto xenbr0
iface xenbr0 inet static
bridge_ports enp4s0
address 192.168.0.xxx
netmask 255.255.0.0
gateway 192.168.0.xxx
dns-nameservers 223.5.5.5
```
重启机器,使得网络生效。   

### 创建HVM虚拟机
HVM是全虚拟化，我们可以用libvirt来管理xen
hypervisor，进而创建HVM虚拟机，步骤如下:   

更改libvirt配置：    

```
# vim /etc/xen/xend-config.sxp 
     xend-unix-server yes
     xend-unix-path /var/lib/xend/xend-socket
# service libvirt-bin restart
libvirt-bin stop/waiting
libvirt-bin start/running, process 5345
# service xen restart
 * Restarting Xen daemons                  [ OK ]
# service xendomains restart
```
最好直接重启整个dom0.    

运行`virt-manager`，添加xen:   

![/images/2016_11_23_17_36_00_276x162.jpg](/images/2016_11_23_17_36_00_276x162.jpg)    

创建虚拟机前，记得先创建出所需的逻辑卷，命令如下:     

```
$ sudo lvcreate -L 5000M -n lvforarchlinux vgonsda
$ ls /dev/vgonsda/lvforarchlinux 
/dev/vgonsda/lvforarchlinux
```
上述命令在vgonsda上创建一块大小为5G的逻辑卷，名字为`lvforarchlinux`.    

创建虚拟机的时候，选择存储设备到这块逻辑盘上，    

![/images/2016_11_23_17_39_05_389x282.jpg](/images/2016_11_23_17_39_05_389x282.jpg)    

接下来的创建方式很common, 安装以后直接用就好。    

### 创建PV虚拟机
PV是半虚拟化，步骤比较繁杂一点，举CentOS6.8的安装为例。   

首先下载`CentOS-6.8-x86_64-minimal.iso`的iso, 提取两个文件用于启动虚拟机安装:    

```
$ sudo mount -t iso9660 -o loop /home/juju/http/CentOS-6.8-x86_64-minimal.iso /mnt
$ cp /mnt/isolinux/vmlinuz /source
$ cp /mnt/isolinux/initrd.img /source
```
创建一个新的pv:    

```
$ sudo lvcreate -L 15000M -n lvforpv vgonsda
```
到`/etc/xen`下，创建一个半虚拟机的定义文件:    

```
$ sudo vim xlcentos.pvlinux
name = "centos"
kernel = "/source/vmlinuz"
ramdisk = "/source/initrd.img"
memory = 1024
vcpus = 1
vif = [ 'bridge=xenbr0' ]
disk = [ '/dev/vgonsda/lvforpv,raw,xvda,rw' ]
```
创建虚拟机:    

```
$ sudo xl create -c /etc/xen/xlcentos.pvlinux
```
在终端安装系统，在Reboot前，需要修改定义文件如下:    

```
$ sudo vim xlcentos.pvlinux
#kernel = "/source/vmlinuz"
#ramdisk = "/source/initrd.img"
bootloader = "/usr/lib/xen-4.6/bin/pygrub"
```
注释掉我们用于启动虚拟机的kernel和ramdisk，使用xen自带的pygrub启动系统，即可从安装好的硬盘启动。    

启动机器:    

```
$ sudo xl create -c /etc/xen/xlcentos.pvlinux
``` 
可以通过`sudo xl list -l`来查看安装好的虚拟机是否是半虚拟的。     

