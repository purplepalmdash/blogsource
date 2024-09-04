+++
title= "OnBuildingUbuntuKernel"
date = "2024-09-04T15:27:17+08:00"
description = "OnBuildingUbuntuKernel"
keywords = ["Technology"]
categories = ["Technology"]
+++
Before building, enable all of the deb-src items.    

Steps:    

```
sudo apt build-dep linux linux-image-unsigned-$(uname -r)
sudo apt install libncurses-dev gawk flex bison openssl libssl-dev dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf llvm
sudo apt install git

apt source linux-image-unsigned-$(uname -r)
chmod a+x debian/rules
chmod a+x debian/scripts/*
chmod a+x debian/scripts/misc/*
fakeroot debian/rules clean
```
Edit the items:    

```
vim debian.xxxx/config/annotations
Change the items you want to change, for example:    
 cat /boot/config-5.4.0-150-generic | grep -i module_force
CONFIG_MODULE_FORCE_LOAD=y
CONFIG_MODULE_FORCE_UNLOAD=y
```
Edit the configurations:    

```
fakeroot debian/rules editconfigs
fakeroot debian/rules binary-headers binary-generic binary-perarch
```
Then after building you could get deb generated.    
