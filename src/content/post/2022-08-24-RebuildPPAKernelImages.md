+++
title= "RebuildPPAKernelImages"
date = "2022-08-24T09:48:47+08:00"
description = "RebuildPPAKernelImages"
keywords = ["Technology"]
categories = ["Technology"]
+++
### ppa kernel
Install ppa kernel on ubuntu20.04 via:    

```
add-apt-repository ppa:tuxinvader/lts-mainline -y
apt update -y
apt-get install -y linux-generic-5.19
```
After reboot, check the kernel via:    

```
# uname -r
5.19.3-051903-generic
```
### Customization of Kernel
uncomment the `deb-src`for ppa:    

```
# cat /etc/apt/sources.list.d/tuxinvader-ubuntu-lts-mainline-focal.list 
deb http://ppa.launchpad.net/tuxinvader/lts-mainline/ubuntu focal main
deb-src http://ppa.launchpad.net/tuxinvader/lts-mainline/ubuntu focal main
# apt update
```
Get the source code:    

```
# apt install linux-source-5.19.3
# ls /usr/src/linux-source-5.19.3/
debian  debian.master  linux-source-5.19.3.tar.bz2
```
Get the build-dep for kernel:    

```
# apt build-dep linux-generic-5.19
# apt build-dep linux-doc
```
Prepare the build tree:    

```
# cd /usr/src/linux-source
# bunzip2 linux-source-5.19.3.tar.bz2
# tar xf linux-source-5.19.3.tar
# mv linux-source-5.19.3/* .
```
Build via:    

```
# fakeroot debian/rules clean
# fakeroot debian/rules editconfigs
# fakeroot debian/rules binary-headers binary-generic binary-perarch
```
build will take sometimes, so drink a cup of H~~~ot tea and wait.   

Check the built result:    

```
# ls /usr/src/*.deb
/usr/src/linux-buildinfo-5.19.3-051903-generic_5.19.3-051903.202208220846_amd64.deb
/usr/src/linux-headers-5.19.3-051903_5.19.3-051903.202208220846_all.deb
/usr/src/linux-headers-5.19.3-051903-generic_5.19.3-051903.202208220846_amd64.deb
/usr/src/linux-image-unsigned-5.19.3-051903-generic_5.19.3-051903.202208220846_amd64.deb
/usr/src/linux-modules-5.19.3-051903-generic_5.19.3-051903.202208220846_amd64.deb
```
### Benchmark
glmark2 result:    

```
# uname -r
5.19.3-051903-generic
# export DISPLAY=:179
# glmark2
=======================================================
    glmark2 2021.02
=======================================================
    OpenGL Information
    GL_VENDOR:     Intel
    GL_RENDERER:   Mesa Intel(R) Graphics (SG1)
    GL_VERSION:    4.6 (Compatibility Profile) Mesa 22.2.0-devel (git-e8fc5cc 2022-06-22 focal-oibaf-ppa)
=======================================================
.........
=======================================================
                                  glmark2 Score: 2886 
=======================================================
```

