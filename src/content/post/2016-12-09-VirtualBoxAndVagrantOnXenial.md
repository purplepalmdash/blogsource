+++
categories = ["Virtualization"]
date = "2016-12-09T10:35:28+08:00"
description = "Install virtualbox on Xenianl"
keywords = ["Virtualization"]
title = "VirtualBoxAndVagrantOnXenial"

+++
Since the vagrant version and virtualbox version are too old in official
repository, we need to upgrade them for using the newest feature, following
are the tips.     

### Virtualbox
#### Manually Install
Download URL:    

[https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)    

Before installing the newest version, make sure you have uninstalled the old
version via `sudo apt-get purge virtualbox*".    

Use dpkg for installing the virtualbox then using `sudo apt-get -f install`
for resolving the dependency issue.     

After installed the virtualbox, use `sudo /sbin/vboxconfig` for building the
kernel module.    

#### Use Repository
In Xenial, use following command for adding the repository:    

```
$ sudo vim /etc/apt/sources.list
 deb http://download.virtualbox.org/virtualbox/debian xenial contrib
$ wget -q https://www.virtualbox.org/download/oracle_vbox_2016.asc -O- | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install -y virtualbox-5.1
```
Also use `sudo /sbin/vboxconfig` for solving the kernel module issue.    

### Vagrant
Download the newest version from:    

[https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)    
