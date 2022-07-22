+++
title= "RebuildQemuForBenchmark"
date = "2022-07-18T09:21:39+08:00"
description = "RebuildQemuForBenchmark"
keywords = ["Technology"]
categories = ["Technology"]
+++
### AIM
Rebuild qemu for enlarge the frame-rate for benchmark GPU, following are the steps.    

### Get the SourceCode
Steps are:    

```
$ sudo apt-get install build-essential pbuilder
$ dpkg -l | grep qemu
### Get the installed package name is `qemu-system-x86`
$ sudo vim /etc/apt/sources.list
$ sudo apt-get update
$ mkdir tmp
$ cd tmp/
$ apt-get source qemu-system-x86
$ ls
qemu-4.2  qemu_4.2-3ubuntu6.23.debian.tar.xz  qemu_4.2-3ubuntu6.23.dsc  qemu_4.2.orig.tar.xz
```
### Code Changes
Refers to `https://www.reddit.com/r/VFIO/comments/nviap2/in_case_your_games_output_60_fps_but_the/`:    

```
$ vim ./qemu-4.2/include/ui/console.h
......
/* in ms */
- #define GUI_REFRESH_INTERVAL_DEFAULT    30
+ #define GUI_REFRESH_INTERVAL_DEFAULT    16
#define GUI_REFRESH_INTERVAL_IDLE     3000
......
```
Rebuild the deb packages, first install the build dependencies for `qemu-system-x86`:    

```
$ sudo apt-get build-dep qemu-system-x86
```
Now Rebuild the package via:      

```
$ cd qemu-4.2/
$ dpkg-buildpackage -rfakeroot -b -uc -us
$ sudo dpkg -l | grep qemu
$ sudo apt-get purge qemu qemu-kvm qemu-system-common qemu-system-gui qemu-system-x86 qemu-utils
$ cd ..
$ ls *.deb
$ sudo dpkg -i *.deb
$ sudo apt-get install -f
```
### Verification
No effect on 60 fps.  
