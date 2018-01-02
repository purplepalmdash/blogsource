+++
date = "2017-05-22T17:00:19+08:00"
categories = ["Virtualization"]
keywords = ["vagrant"]
description = "Expand vagrant's disks"
title = "AddSecondDiskForVagrant"

+++
### Purpose
For adding second disk to vagrant machine, while vagrant machine is running
under virtualbox hypervisor. Following are the steps for doing this.    

### Steps
`Vagrantfile` definition:    

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|

  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    #vb.gui = true
      unless File.exist?('./secondDisk.vdi')
        vb.customize ['createhd', '--filename', './secondDisk.vdi', '--variant', 'Fixed', '--size', 10 * 1024]
      end
  
    # Customize the amount of memory on the VM:
    vb.memory = "1024"
    vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', './secondDisk.vdi']

  end
end

```
The initial.sh should be run under the vagrant machine has been created:    

```
cat initial.sh
set -e
set -x
echo 'Acquire::http::Proxy "http://192.168.0.121:3142";'>/etc/apt/apt.conf.d/01proxy
# Added the primary repository
apt-get update
apt-get -y install vim lvm2

if [ -f /etc/disk_added_date ]
then
   echo "disk already added so exiting."
   exit 0
fi


sudo fdisk -u /dev/sdb <<EOF
n
p
1


t
8e
w
EOF

pvcreate /dev/sdb1
vgextend vagrant-vg /dev/sdb1
lvextend -l +100%FREE /dev/vagrant-vg/root
resize2fs /dev/vagrant-vg/root

date > /etc/disk_added_date
```

### !!!Notice!!!
You should especially notice the lvextend's configuration, for every vagrant box 
has the different naming mechanism.    
