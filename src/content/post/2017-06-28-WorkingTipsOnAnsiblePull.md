+++
keywords = ["Ansible"]
description = "WorkingTipsOnAnsiblePull"
title = "WorkingTipsOnAnsiblePull"
date = "2017-06-28T15:05:08+08:00"
categories = ["Linux"]

+++
### Environment Preparation
Create the vagrant environment via following Vagrantfile:    

```
# -*- mode: ruby -*-
# vi : set ft=ruby :

Vagrant.configure(2) do |config|

  config.ssh.insert_key = false # Use the same insecure key provided from box for each machine
  config.vm.box = "UbuntuTrusty3G" 
  config.vm.provision :shell, path: "initial.sh"
  config.vm.box_check_update = false # do not check for updates  ( not recommended , just for demo )
  config.vm.boot_timeout = 700
  #config.hostmanager.enabled = true
  config.hostmanager.enabled = false
  config.hostmanager.ignore_private_ip = false
   
   N = 2
   (1..N).each do |i| # do for each server i
     config.vm.define "testnode#{i}" do |config|  # do on node i
     config.vm.hostname = "testnode#{i}"
     puts " testnode#{i}  "
     config.vm.network "private_network", ip: "192.168.56.3#{i}"
     config.vm.provider "virtualbox" do |v| # virtual box configuration
       v.memory = "256"
       v.cpus = 1
        if ! File.exist?("testnode#{i}_disk_a.vdi") # create disks only once 
           v.customize ['createhd', '--filename', "testnode#{i}_disk_a.vdi", '--size', 8192 ]
           #v.customize ['createhd', '--filename', "testnode#{i}_disk_b.vdi", '--size', 8192 ]
           v.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', "testnode#{i}_disk_a.vdi"]
           #v.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', "testnode#{i}_disk_b.vdi"]
        end #  create disks only once 
     end # end virtual box configuration

     end # do on node i
   end # for each server i

end  # end Vagrant.configure(2)
```
The initial.sh could be refered to previous post, for extending the 3G's disk into 11G(3G+8G).   

### Package Preparation
Via following commands:    
Common packages(cron is created by default):    

```
# apt-get -y update
# apt-get install -y git cron sshpass
``` 
lb packages:   

```
# apt-get install -y haproxy socat
```
web packages:    

```
# apt-get install -y nginx
```

mgmt node:    

```
# apt-add-repository -y ppa:ansible/ansible
# apt-get install -y ansible
```

OK, all of the deb packages are available, so collect them and create your own deb repository via following command:    

```
# mkdir -p /root/pkgs/
# cd /var/cache/apt/archives/
# find . | grep deb$ | xargs -I % cp % /root/pkgs/
# dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
```
Now upload the folder of `root/pkgs` to your web server, you local repository is OK now.    
Be sure to ignore authentification of pkgs:    

```
# vim /etc/apt/apt.conf.d/99myown
APT::Get::AllowUnauthenticated "true";
# vim /etc/apt/sources.list
deb http://192.168.0.220/UbuntuTrusty3G/ /
```
