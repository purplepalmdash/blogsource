+++
title= "LXCBasedAndroid"
date = "2021-09-02T08:09:41+08:00"
description = "LXCBasedAndroid"
keywords = ["Technology"]
categories = ["Technology"]
+++
### References
Refers to :    
[https://github.com/elliott-wen/anbox-direct-gpu-access](https://github.com/elliott-wen/anbox-direct-gpu-access)    

This project could run android in lxc, with a modified UI for accessing the android UI.    

### Environment
Hardware and OS information is listed as:    

```
# cat /proc/cpuinfo | grep 'model name'
model name	: Intel(R) Core(TM) i5-8265UC CPU @ 1.60GHz
$ free -m
              total        used        free      shared  buff/cache   available
Mem:          15765         336       14602         182         826       14960
Swap:          4095           0        4095
$ cat /etc/issue
Ubuntu 18.04.5 LTS \n \l
```

### Steps
Initialize the environment via:    

```
$ sudo apt-get install -y 
$ sudo apt-get upgrade -y
$ sudo apt-get install -y lubuntu-desktop
$ sudo apt-get install lxc  uidmap dkms
$ sudo usermod --add-subuids 100000-165536 dash
$ sudo usermod --add-subgids 100000-165536 dash
$ sudo chmod +x $HOME
$ cd ~/.config/
$ mkdir lxc
$ cd lxc/
$ vim default.conf
lxc.net.0.type = veth
lxc.net.0.link = lxcbr0
lxc.net.0.flags = up
lxc.net.0.hwaddr = 00:16:3e:xx:xx:xx
lxc.idmap = u 0 100000 65536
lxc.idmap = g 0 100000 65536
$ sudo vim /etc/lxc/lxc-usernet
  # USERNAME TYPE BRIDGE COUNT
  dash	veth lxcbr0 10
$ git clone https://github.com/anbox/anbox-modules.git
$ cd anbox-modules
$  ./INSTALL.sh
$ sudo reboot
$ mkdir -p /home/dash/emugui/disk
$ mkdir -p /home/dash/emugui/disk/data
$ mkdir -p /home/dash/emugui/disk/cache
$ git clone https://github.com/elliott-wen/anbox-direct-gpu-access.git
$ cd anbox-direct-gpu-access
$ sudo apt-get install -y clang libxcb1-devel libx11-xcb-dev libxcb-xinput-dev libxcb-present-dev libxcb-dri3-dev libxcb-icccm4-dev libpulse-dev 
$ ./build.sh
$ cd ~/.local/share/lxc
$ lxc-create -t busybox -n android
$ cd android
$ mv rootfs/ rootfs.back
$ tar xvf ~/rootfs.tar -C .
$ sudo /home/dash/anbox-direct-gpu-access-master/nsexex -b ~/.local/share/lxc/android/rootfs 0 100000 65536
$ mv config config.back
$ vim config
```
The config file for lxc is listed as:    

```
# Template used to create this container: /usr/share/lxc/templates/lxc-busybox
# Parameters passed to the template:
# Template script checksum (SHA-1): 21abc1440b73cdb95d96d5459b27c3a87df9976f
# For additional config options, please look at lxc.container.conf(5)
lxc.include = /etc/lxc/default.conf
lxc.idmap = u 0 100000 65536
lxc.idmap = g 0 100000 65536
#lxc.rootfs.path = dir:/home/elliott/.local/share/lxc/android/rootfs
lxc.rootfs.path = dir:/home/dash/.local/share/lxc/android/rootfs
lxc.mount.entry = /home/dash/emugui/disk/data data none bind,optional 0 0
lxc.mount.entry = /home/dash/emugui/disk/cache cache none bind,optional 0 0
lxc.mount.entry = /dev/dri/card0 dev/dri/card0 none bind,optional,create=file 0 0
lxc.mount.entry = /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file 0 0	
lxc.mount.entry = /dev/binder dev/binder none bind,optional,create=file 0 0
lxc.mount.entry = /dev/uinput dev/uinput none bind,optional,create=file 0 0
lxc.mount.entry = /dev/ashmem dev/ashmem none bind,optional,create=file 0 0
lxc.mount.entry = /tmp/android-dbus host none bind,optional,create=dir 0 0
lxc.mount.entry = /tmp/android-dbus/input dev/input none bind,optional,create=dir 0 0
lxc.mount.entry = /dev/fuse dev/fuse none bind,optional,create=file 0 0
lxc.signal.halt = SIGUSR1
lxc.signal.reboot = SIGTERM
lxc.uts.name = "android"
lxc.tty.max = 0
lxc.pty.max = 1024
lxc.tty.dir = ""
lxc.net.0.type="veth"
lxc.net.0.flags="up"
lxc.net.0.link="lxcbr0"


# When using LXC with apparmor, uncomment the next line to run unconfined:
lxc.apparmor.profile = unconfined
lxc.mount.auto = proc:mixed sys:mixed cgroup:mixed
lxc.autodev = 1
lxc.environment = PATH=/system/bin:/system/sbin:/system/xbin:/bin
lxc.init.cmd=/init
```
Change the mode for dev files:    

```
sudo chmod 0600 -R /dev/binder  /dev/ashmem  /dev/dri/*
```
Then start the lxc instance via:    

```
lxc-start -F -n android
```
### adb connection
Install adb via:    

```
# sudo apt-get install -y adb
# adb root
# adb connect 10.0.3.174
# adb  shell
# adb push ~/F-Droid.apk
# adb push ..... ....
```

Stop the intance:   

```
# lxc -stop -n android -k
```

