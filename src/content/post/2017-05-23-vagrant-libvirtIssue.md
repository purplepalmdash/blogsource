+++
title = "vagrant-libvirt issue on ArchLinux"
date = "2017-05-23T16:54:24+08:00"
categories = ["Virtualization"]
keywords = ["vagrant"]
description = "vagrant-libvirt issue"

+++
Previously install vagrant-libvirt is a tough task on ArchLinux, thus I do
following steps to let my vagrant-libvirt running again on archlinux, steps
are listed as following:    

Remove the installed vagrant and backup the configuration files.    

```
$ sudo pacman -Rsn vagrant
$ sudo mv ~/.vagrant.d ~/vagrant.d.back
$ sudo rm -rf /opt/vagrant
```

Install the vagrant-libvirt in AUR repository:    

```
$ yaourt vagrant-libvirt
$ tsocks vagrant plugin install vagrant-libvirt
$ vagrant plugin list
vagrant-libvirt (0.0.40)
```
Please notice the plugin should also be installed after you have installed the
vagrant-libvirt via yaourt.    
