+++
title = "MakeCoreOSInstallLiveCD"
date = "2019-08-22T11:07:02+08:00"
description = "MakeCoreOSInstallLiveCD"
keywords = ["Linux"]
categories = ["Linux"]
+++
### AIM
livecd->coreos_install in livecd->Reboot to coreos
### Steps
Install cubic on ubuntu 18.04.3:    

```
sudo apt-add-repository ppa:cubic-wizard/release
sudo apt update
sudo apt-get install -y cubic
```
Download the `ubuntu-18.04.3-desktop-amd64.iso`, and check its md5sum:    

```
$ md5sum ubuntu-18.04.3-desktop-amd64.iso 
72491db7ef6f3cd4b085b9fe1f232345  ubuntu-18.04.3-desktop-amd64.iso
```
Making working directory:    

Changes: I found the coreos install iso could do the same thing, ignored.    
