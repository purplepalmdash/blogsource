+++
title= "vagranttips"
date = "2023-11-15T08:36:39+08:00"
description = "vagranttips"
keywords = ["Technology"]
categories = ["Technology"]
+++
Vagrantfile definition:    

```
Vagrant.configure("2") do |config|
  config.vm.box = "rockylinux/9"
config.disksize.size = '80GB'

config.vm.provider "virtualbox" do |v|
  v.memory = 81920
  v.cpus = 6
end

end
```
Should install following plugins:    

```
$ vagrant plugin list
vagrant-disksize (0.1.3, global)
vagrant-libvirt (0.7.0, system)
vagrant-vbguest (0.31.0, global)
```
Install via `vagrant plugin install xxxxxxxxxx`    
Resize in vm:    

```
cfdisk /dev/sda
xfs_growfs /dev/sda5
```

On Ubuntu lvm, if you resize the disk, do following:    

```
lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv 
resize2fs /dev/ubuntu-vg/ubuntu-lv 
```
