+++
title= "WorkingtipsOnXenGT"
date = "2022-09-28T16:14:39+08:00"
description = "WorkingtipsOnXenGT"
keywords = ["Technology"]
categories = ["Technology"]
+++
### hardware
hardware:    

```
$ cat /proc/cpuinfo | grep model
model		: 60
model name	: Intel(R) Core(TM) i5-4460  CPU @ 3.20GHz
$ free -g
             total       used       free     shared    buffers     cached
Mem:            30          1         29          0          0          0
-/+ buffers/cache:          0         30
Swap:            0          0          0
dash@ubuntu1404:~$ cat /etc/issue
Ubuntu 14.04.5 LTS \n \l

dash@ubuntu1404:~$ uname -a
Linux ubuntu1404 4.4.0-31-generic #50~14.04.1-Ubuntu SMP Wed Jul 13 01:07:32 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux

```

Install ubuntu14.04, change repository to `mirrors.ustc.edu.cn`, then install packages:    

```
# apt-get install libarchive-dev libghc-bzlib-dev  zlib1g-dev mercurial gettext bcc iasl uuid-dev libncurses5-dev kpartx bc libperl-dev libgtk2.0-dev libc6-dev-i386 libaio-dev libsdl1.2-dev nfs-common libyajl-dev libx11-dev autoconf libtool xsltproc bison flex xutils-dev xserver-xorg-dev x11proto-gl-dev libx11-xcb-dev vncviewer libxcb-glx0 libxcb-glx0-dev libxcb-dri2-0-dev libxcb-xfixes0-dev bridge-utils python-dev bin86 git vim libssl-dev pciutils-dev tightvncserver ssh texinfo -y
```


### Check
Before install driver:    

![/images/2022_09_29_10_00_16_732x530.jpg](/images/2022_09_29_10_00_16_732x530.jpg)

Install driver:    

![/images/2022_09_29_10_06_34_519x420.jpg](/images/2022_09_29_10_06_34_519x420.jpg)

Restart the machine, check the driver status:    


