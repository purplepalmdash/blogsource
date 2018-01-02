+++
title = "创建Vagrant Box"
categories = ["Linux"]
keywords = ["Vagrant"]
date = "2017-02-20T10:59:17+08:00"
description = "Create vagrant box from scratch"

+++
### Aims
Recently I am frequently using vagrant for verification, so quickly creating
the vagrant box becomes the essential tasks. Previsouly I am suing packer for
building vagrant box, but it requires you for writing the complex
configuration files. How to quickly generate the vagrant box using virtualbox,
following lists the steps
### Virtual Machine
Create a new virtual machine in virtualbox, make sure you locate your disk
file in `/tmp` folder!!! In ArchLinux, the `/tmp` directly is actually the
ramfs, so the read/write speed would be pretty quick! Thus you could greatly
save your time for installing the system.    

The partition size depends on your own requirement, I have a 8G memory
machine, So I choose 3G for partition disk, later we will see the partition
will be seperated into 2 part(Swap/Root), the swap is equal to your virtual
machine memory size, all of the remain spaces will be allocated into root
directory.    

In partition, you should select `Use all spaces and setup LVM`, the
installation process will automatically create the layout for you.    

Create the username/password for `vagrant/vagrant`, also your hostname would
also be `vagrant`.    

Enable the sshd.    

Finish installation and reboot your machine.     

### Configuration
Update and upgrade your system via:    

```
$ sudo apt-get update -y
$ sudo apt-get upgrade -y
# Restart the machine
$ sudo shutdown -r now
```
Add vagrant user to sudoer's list:    

```
$ sudo su -
$ visudo
# Add the following line to the end of the file.
vagrant ALL=(ALL) NOPASSWD:ALL
```
Install the vagrant publicfile:    

```
$ mkdir -p /home/vagrant/.ssh
$ wget --no-check-certificate https://raw.github.com/mitchellh/vagrant/master/keys/vagrant.pub -O /home/vagrant/.ssh/authorized_keys
# Ensure we have the correct permissions set
$ chmod 0700 /home/vagrant/.ssh
$ chmod 0600 /home/vagrant/.ssh/authorized_keys
$ chown -R vagrant /home/vagrant/.ssh
```
Install the guest tools:    

```
$ sudo apt-get install -y gcc build-essential linux-headers-server
```
Mount the guest tools, and do following commands for compiling the
guest-tools:    

```
$ sudo mount /dev/sr0 /mnt
$ cd /mnt
$ sudo ./VBoxLinuxAdditions.run
```
Before your packaging the box, remove the un-continous spaces in disk via
following command:    

```
$ sudo dd if=/dev/zero of=/EMPTY bs=1M
$ sudo rm -f /EMPTY
# Shutdown the machine
$ sudo shutdown -h now
```

### Packaging the Box
Packaging is so simple via:    

```
$ vagrant package --base <VitualBox VM Name>
```
You could install it via:    

```
$ vagraant box add package.box --name "YourName"
```

### Exapand your disks
An example of Vagrantfile is listed as following:    

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "UbuntuTrusty3G"
  config.vm.provision :shell, path: "initial.sh"

  config.vm.provider "virtualbox" do |vb|
      unless File.exist?('./secondDisk.vdi')
        vb.customize ['createhd', '--filename', './secondDisk.vdi', '--variant', 'Fixed', '--size', 10 * 1024]
      end
  
    vb.memory = "1024"
    vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', './secondDisk.vdi']
  end
end
```
The `initial.sh` called in  Vagrantfile is like following:    

```
set -e
set -x
#echo 'Acquire::http::Proxy "http://192.168.0.121:3142";'>/etc/apt/apt.conf.d/01proxy
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
This script will automatically expand your root disk from the original 2.5G to 12.5G, you could adjust the size in Vagrantfile.    
