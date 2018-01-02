+++
title = "BuildDebianDockerImageLocally"
date = "2017-08-29T09:21:07+08:00"
description = "BuildDEbianDockerImageLocally"
keywords = ["Linux"]
categories = ["Linux"]
+++
### AIM
Recently I am working in a network isolated environment, this means, I am
unable to connect to internet freely. But I have to finish my working using
some specific docker images, how? So I have to build them locally using iso.    

### Preparation
Download debian dvds from internet and tranfer them into the intranet:    

```
# ls -l debian*
-rw-r--r-- 1 dash root 3.6G Jul 21 00:59 debian-9.0.0-amd64-DVD-1.iso
-rw-r--r-- 1 dash root 4.4G Jul 21 01:57 debian-9.0.0-amd64-DVD-2.iso
-rw-r--r-- 1 dash root 4.4G Jul 21 02:52 debian-9.0.0-amd64-DVD-3.iso
```
Install debian in virtualbox using debian-9.0.0-amd64 DVD1 iso. Using basic
desktop with sshd server.    

### Repository
Change the repository with following command:   
```
# mount -t iso9660 -o loop /media/sdb/iso/debian-9.0.0-amd64-DVD-1.iso /media/sdb/debianrepo/cd1/
# mount -t iso9660 -o loop /media/sdb/iso/debian-9.0.0-amd64-DVD-2.iso /media/sdb/debianrepo/cd2/
# mount -t iso9660 -o loop /media/sdb/iso/debian-9.0.0-amd64-DVD-3.iso /media/sdb/debianrepo/cd3/
```
While the directory `/media/sdb/debianrepo/` is the directory under the web server. Thus we could change the apt's source file pointint to this webserver.    

```
# vim /etc/apt/sources.list
deb http://192.168.56.1/debianrepo/cd1/	stable main contrib
deb http://192.168.56.1/debianrepo/cd2/	stable main contrib
deb http://192.168.56.1/debianrepo/cd3/	stable main contrib
```
Enable the Unauthenticated installation, and install vim for testing:    

```
# vim /etc/apt/apt.conf.d/99myown 
APT::Get::AllowUnauthenticated "true";
# apt-get update
# apt-get -y install vim
```

### BootStrap
Standard Bootstrap won't be used, so we have to use a tool called `demoostrap`.    
Download position:    

```
# git clone https://github.com/jwilk/demoostrap.git
```
Install some necessray packages:    

```
# apt-get install -y python3-debian apt-utils debootstrap dpkg-repack
```
demoostrap the basic filesystem via:    

```
# mkdir docker_root
# ./demoostrap docker_root/
# du -hs docker_root/
179M	docker_root/
```
In next step we will chroot into this new filesystem and install some necessary package. 

### chroot
The filesystem under `docker_root` only contains basic filesystem. In order to work properly we have to install at least minimum `apt-get`, so following are the steps:    

```
# mount -t devpts devpts docker_root/dev/pts
# mount -t proc proc docker_root/proc
# mount -t sysfs sysfs docker_root/sys
```
Before chroot into the system we have to download some packages manually into the `docker_root` folder, because in chrooted system we could only use dpkg for installation.    

```
# mkdir -p docker_root/install
# cd docker_root/install
# wget http://192.192.192.91/debianrepo/cd1/debian/debian/pool/main/a/apt/apt_1.4.6_amd64.deb
# wget http://192.192.192.91/debianrepo/cd1/debian/pool/main/a/apt/libapt-pkg5.0_1.4.6_amd64.deb
# wget http://192.192.192.91/debianrepo/cd1/debian//pool/main/g/gcc-6/libstdc++6_6.3.0-18_amd64.deb
# wget http://192.192.192.91/debianrepo/cd1/debian/pool/main/a/adduser/adduser_3.115_all.deb
# wget http://192.192.192.91/debianrepo/cd1/debian/pool/main/d/debian-archive-keyring/debian-archive-keyring_2017.5_all.deb
# wget http://192.192.192.91/debianrepo/cd1/debian//pool/main/g/gnupg2/gpgv_2.1.18-6_amd64.deb
```
Now chroot in, and manually install these packages:    

```
# chroot docker_root/
#  cd /install
# dpkg -i gpgv_2.1.18-6_amd64.deb 
# dpkg -i libstdc++6_6.3.0-18_amd64.deb 
# dpkg -i libapt-pkg5.0_1.4.6_amd64.deb 
# dpkg -i debian-archive-keyring_2017.5_all.deb 
# dpkg -i adduser_3.115_all.deb 
# dpkg -i apt_1.4.6_amd64.deb
# which apt-get
/usr/bin/apt-get
```
### Further Modification
In host machine, we manually change the `sources.list` file and 
Now we could exit the chroot system, and tar `docker_root` directory, 

```
# vim etc/apt/sources.list
deb http://192.168.56.1/debianrepo/cd1/	stable main contrib
deb http://192.168.56.1/debianrepo/cd2/	stable main contrib
deb http://192.168.56.1/debianrepo/cd3/	stable main contrib
```
Enable the Unauthenticated installation:    

```
# vim etc/apt/apt.conf.d/99myown 
APT::Get::AllowUnauthenticated "true";
```

### zip and import
tar the directory using following command:    

```
# tar czvf docker_root.tar.gz docker_root/
```
Then in the untared directory you could import it into docker repository via:    

```
# tar -c * | docker import - myowndebian:raw
# docker images | grep myowndebian
myowndebian                                                    raw                 909759ad7fad        23 seconds ago      189MB
# docker run -it myowndebian:raw /bin/bash
root@f95e7f8d6427:/# ls
bin   dev  home     lib    media  opt	root  sbin  sys  usr
boot  etc  install  lib64  mnt	  proc	run   srv   tmp  var
```
At this point, you have a selt-built docker box, from this start point you could add more packages into your docker images and form your own images offline. Just enjoy it!  
