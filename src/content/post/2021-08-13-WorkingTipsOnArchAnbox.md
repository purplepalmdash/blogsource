+++
title= "WorkingTipsOnArchAnbox"
date = "2021-08-13T16:29:43+08:00"
description = "WorkingTipsOnArchAnbox"
keywords = ["Technology"]
categories = ["Technology"]
+++
### VM Prepration
Download qcow2 from gitlab(arch built), then start a vm.  
Update the mirrorlist from `https://archlinux.org/mirrorlist/`
### Steps
Install yay:    

```
$ sudo pacman -Sy git fakeroot vim pacman-mirrorlist
$ sudo pacman -Sy base-devel
$ cd /opt
$ sudo git clone https://aur.archlinux.org/yay-git.git
$ sudo chown -R  arch:arch ./yay-git/
$ cd yay-git
$ makepkg -si
```
Install `linux-mainline`:    

```
$ yay linux-mainline
```
