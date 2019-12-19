+++
title = "WorkingTipsOnKubesprayKongFuZi"
date = "2019-12-11T08:35:55+08:00"
description = "WorkingTipsOnKubesprayKongFuZi"
keywords = ["Linux"]
categories = ["Linux"]
+++
### 目的
Kubespray在离线环境下，完全不考虑包管理、docker升级的发行版。    
### 技术要点
1. 离线情况下的源仓库准备。    
2. 完全离线情况下ansible的执行。   

### 环境准备(与本文无关)
Ubuntu 16.04.2， 最小化安装后，做成vagrant box:      

```
$ sudo vim /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
$ sudo grub-mkconfig -o /boot/grub/grub.cfg
$ sudo useradd -m vagrant
$ sudo passwd vagrant
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
$ sudo mkdir -p /home/vagrant/.ssh
$ sudo chmod 0700 /home/vagrant/.ssh/
$ sudo vim /home/vagrant/.ssh/authorized_keys
$ sudo cat /home/vagrant/.ssh//authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ== vagrant insecure public key
$ sudo chown -R vagrant /home/vagrant/.ssh
$ sudo cp /home/test/.bashrc /home/vagrant/.bashrc 
$ sudo cp /home/test/.bash_logout /home/vagrant/.bash_logout
$ sudo cp /home/test/.profile /home/vagrant/.profile
$ sudo vim /home/vagrant/.profile 
add
[ -z "$BASH_VERSION" ] && exec /bin/bash -l
$ sudo chsh -s /bin/bash vagrant
$ sudo  vim /etc/ssh/sshd_config 
AuthorizedKeysFile .ssh/authorized_keys
$ sudo visudo -f /etc/sudoers.d/vagrant
vagrant ALL=(ALL) NOPASSWD:ALL
Defaults:vagrant !requiretty
$ sudo vim /etc/network/interfaces
change from ens3 to eth0
auto eth0
inet .....
```
关闭机器后，缩减磁盘空间编辑vagrantfile文件并最终创建box:     

```
$ sudo qemu-img convert -c -O qcow2  ubuntu160402.qcow2 ubuntu160402Shrink.qcow2
$ sudo vim metadata.json
{
"provider"     : "libvirt",
"format"       : "qcow2",
"virtual_size" : 80
}
$ sudo vim Vagrantfile
Vagrant.configure("2") do |config|
       config.vm.provider :libvirt do |libvirt|
       libvirt.driver = "kvm"
       libvirt.host = 'localhost'
       libvirt.uri = 'qemu:///system'
       end
config.vm.define "new" do |custombox|
       custombox.vm.box = "custombox"
       custombox.vm.provider :libvirt do |test|
       test.memory = 1024
       test.cpus = 1
       end
       end
end
$ sudo tar cvzf custom_box.box ./metadata.json ./Vagrantfile ./box.img
```
添加并检查box是否可用:      

```
$ vagrant box add custom_box.box --name "ubuntu160402old"
$ vagrant init ubuntu160402old
$ vagrant up --provider=libvirt
```
### Server实现
沿用coreos的机制，将docker/docker-compose以二进制的方式安装。安装完毕后通过容器启动几乎所有的服务:    

```
ntp
harbor
ansible
dnsmasq
fileserver
```

