+++
title= "BuildingCelanda"
date = "2024-03-11T09:31:02+08:00"
description = "BuildingCelanda"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Environment
Software/OS:    

```
dash@buildCeladonUbuntu180406:~$ uname -a
Linux buildCeladonUbuntu180406 5.4.0-84-generic #94~18.04.1-Ubuntu SMP Thu Aug 26 23:17:46 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
dash@buildCeladonUbuntu180406:~$ cat /etc/issue
Ubuntu 18.04.6 LTS \n \l
```
Install packages:    

```
sudo apt install -y build-essential uuid-dev iasl git gcc-5 nasm unzip python3-distutils-extra python-distutils-extra libpixman-1-dev libssl-dev vim socat libsdl1.2-dev libspice-server-dev autoconf libtool xtightvncviewer tightvncserver x11vnc uuid-runtime uuid uml-utilities python-dev liblzma-dev libc6-dev libegl1-mesa-dev libdrm-dev libgbm-dev spice-client-gtk libegl1-mesa-dev libgtk2.0-dev libusb-1.0-0-dev libepoxy-dev libaio-dev libgtk-3-dev ovmf libsdl2-dev build-essential net-tools bridge-utils openssh-server openssh-client bison flex libelf-dev libncurses-dev  git libfdt-dev git-lfs xorriso pkg-config python-pystache python3-pystache python3-pip cmake

apt install -y ninja-build ncurses-term  llvm libffi-dev keyutils gawk curl
apt install -y rsync mtools dosfstools sbsigntool zip kmod sudo python3-mako
```
config git:     

```
# cat ~/.gitconfig 
[user]
	email = root@gmail.com
	name = root
[color]
	ui = auto
[http]
	proxy = socks5://192.168.1.7:10808
[https]
	proxy = socks5://192.168.1.7:10808
```
Configure python:    

```
apt install -y python-six
ln -sf /usr/bin/python3 /usr/bin/python
```
Get repo:     

```
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/.bin/repo
$ sudo cp ~/.bin/repo /usr/bin/repo
$ sudo chmod 777 /usr/bin/repo
$ repo --version
<repo not installed>
repo launcher version 2.42
       (from /usr/bin/repo)
git 2.17.1
Python 3.6.9 (default, Jan 26 2021, 15:33:00) 
[GCC 8.4.0]
OS Linux 5.4.0-84-generic (#94~18.04.1-Ubuntu SMP Thu Aug 26 23:17:46 UTC 2021)
CPU x86_64 (x86_64)
Bug reports: https://issues.gerritcodereview.com/issues/new?component=1370071
```
Install meson:    

```
$ sudo su
# python3 -m pip install meson
```
Install `glslang`:      

```
git clone https://github.com/KhronosGroup/glslang.git
cd glslang/
git checkout 7.10.2984
python update_glslang_sources.py 
mkdir build
cd build
cmake ..
make
sudo make install
```

### Using docker
via:    

```
sudo docker run -it -v /media/sda/nfs_share/buildinDocker:/buildinDocker ubuntu:18.04 /bin/bash
```
Building command:     

```
$ repo sync -l -j16
$ repo forall -c git lfs pull
$ source build/envsetup.sh
$ lunch caas-userdebug
$ time make flashfiles BASE_LTS2020_YOCTO_KERNEL=true -j16
......
#### build completed successfully (01:56:38 (hh:mm:ss)) ####


real	116m37.484s
user	1589m56.841s
sys	86m11.933s

```
