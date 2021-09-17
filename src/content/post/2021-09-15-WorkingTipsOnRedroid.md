+++
title= "WorkingTipsOnRedroid"
date = "2021-09-15T08:54:22+08:00"
description = "WorkingTipsOnRedroid"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Sys Info
idv hardware, Ubuntu 18.04 server:    

```
Intel(R) Core(TM) i5-8265UC CPU @ 1.60GHz
16 G Memory
256 G nvme ssd
dash@redroid:~$ cat /etc/issue
Ubuntu 18.04.5 LTS \n \l
dash@redroid:~$ uname -r
4.15.0-156-generic
```
### Kernel Preparation
Kernel module preparation:   

```
$ sudo apt-get upgrade -y
$ sudo reboot
$ sudo apt-get install -y dkms linux-headers-generic
$ mkdir Code
$ cd Code && git clone https://github.com/remote-android/redroid-modules.git
$ cd redroid-modules/
$ sudo cp redroid.conf /etc/modprobe.d/
$ sudo cp 99-redroid.rules /lib/udev/rules.d/
$ sudo cp -rT ashmem/ /usr/src/redroid-ashmem-1
$ sudo cp -rT binder /usr/src/redroid-binder-1
$ sudo dkms install redroid-ashmem/1
$ sudo dkms install redroid-binder/1
```
Check via:    

```
dash@redroid:~$ grep binder /proc/filesystems
nodev	binder
dash@redroid:~$ grep ashmem /proc/misc 
 55 ashmem
```
### Docker
Install docker via:    

```
$ sudo apt-get install -y docker.io
$ sudo systemctl start docker
$ sudo systemctl enable docker
```
Prepare the image via `docker pull xxxx`:   

```
$ docker images
REPOSITORY        TAG            IMAGE ID       CREATED       SIZE
redroid/redroid   12.0.0-amd64   3000c3e2a297   5 weeks ago   1.52GB
redroid/redroid   9.0.0-latest   a38ac26defd9   5 weeks ago   1.55GB
```
### Start Android Docker
Start 2 android docker session via:   

```
docker  run -itd --rm --memory-swappiness=0 --privileged     --pull always     -v ~/data:/data     -p 5555:5555     redroid/redroid:9.0.0-latest
docker run -itd --rm --memory-swappiness=0 --privileged     --pull always     -v ~/data1:/data -p 5556:5555 redroid/redroid:12.0.0-amd64
```

Connect(Archlinux client):      

```
$ yay scrcpy
$ adb connect 192.168.1.119:5555
$ adb connect 192.168.1.119:5556
$ scrcpy --serial 192.168.1.119:5555
$ scrcpy --serial 192.168.1.119:5556
```

![/images/2021_09_15_09_25_02_583x1052.jpg](/images/2021_09_15_09_25_02_583x1052.jpg)

