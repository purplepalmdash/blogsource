+++
date = "2017-02-17T11:09:33+08:00"
categories = ["Linux"]
keywords = ["Linux"]
description = "Configuration of vagrant"
title = "Configuraiton For Vagrant"

+++
### Vagrantfile
Configuration for setting Ubuntu14.04, bridged networking, and setting its
routing to specified node.    

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure("2") do |config|
  config.vm.box = "minimum/ubuntu-trusty64-docker"

  # Networking
  config.vm.network "public_network", ip:"192.168.10.217", netmask: "16",bridge:"xenbr0"
  # default router
  config.vm.provision "shell",
    run: "always",
    inline: "route add default gw 192.168.0.176"

  # delete default gw on eth0
  config.vm.provision "shell",
    run: "always",
    inline: "eval `route -n | awk '{ if ($8 ==\"eth0\" && $2 != \"0.0.0.0\") print \"route del default gw \" $2; }'`"
  
  # Hostname
  config.vm.hostname = "CSMgmt"

  # Memory and CPU Configuration
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    # vb.gui = true
  
    # Customize the amount of memory on the VM:
    vb.memory = "4096"
    vb.cpus = 2
  end

end
```
