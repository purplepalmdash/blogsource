+++
title= "OfflineArm64DesktopEnvSetup"
date = "2020-08-27T17:19:40+08:00"
description = "OfflineArm64DesktopEnvSetup"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Download via docker
Run a docker instance via:    

```
$ sudo docker run -v /mnt:/mnt -it ubuntu:focal-20200115 /bin/bash
```

In docker instance, do following:   

```
rm -f /etc/apt/apt.conf.d/docker-clean
sed -i 's/ports.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
cd /mnt
apt-get -d -o dir::cache=`pwd` -o Debug::NoLocking=1 install xubuntu-desktop xubuntu-core chromium-browser  firefox xrdp virt-manager ubuntu-wallpapers xubuntu-community-wallpapers xubuntu-community-wallpapers-focal xubuntu-wallpapers lxd lxc
apt-get install -y snapd
snap download lxd
snap download chromium

```
### Transfer
Transfer these packages into the offline environments, and do following:    

```
# cd ~/pkgs
# dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
```
### Install
In a ubuntu base environment, do following:    

```
# vim /etc/apt/sources.list
deb [trusted=yes] file:///home/test/focal/ ./
# apt-get update -y
# apt-get install -y  xubuntu-desktop xubuntu-core  firefox xrdp virt-manager ubuntu-wallpapers xubuntu-community-wallpapers xubuntu-community-wallpapers-focal xubuntu-wallpapers
```

### Configure xrdp
Configure xrdp via:   

```
$ sudo systemctl status xrdp
$ sudo adduser xrdp ssl-cert  
$ sudo systemctl restart xrdp
$ sudo ufw disable
$ echo xfce4-session >~/.xsession
$ sudo vim /etc/xrdp/startwm.sh
#!/bin/sh

if [ -r /etc/default/locale ]; then
  . /etc/default/locale
  export LANG LANGUAGE
fi

startxfce4
$ sudo systemctl restart xrdp
```
So now you could use xfce4 as your remote desktop to linux. 

### snapd installation
In docker run:    

```
# sudo apt-get install -y snapd
# snap download lxd
# snap download chromium
# snap download gtk-common-themes
# snap download core
# snap download core18
```
Install sequence:    

```
# snap install  (core/core18/gtk-common-themes/lxd/chromium)
chromium_1253.snap  core18_1888.snap  core_9806.snap                 gtk-common-themes_1506.snap  lxd_16946.snap
chromium_1253.assert  core18_1888.assert  core_9806.assert  gtk-common-themes_1506.assert  lxd_16946.assert   
```
