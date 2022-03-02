+++
title= "RunAVDInCentOS76Docker"
date = "2022-03-01T11:20:00+08:00"
description = "RunAVDInCentOS76Docker"
keywords = ["Technology"]
categories = ["Technology"]
+++
步骤：   

```
# docker pull centos:7.6.1810
```
挂载iso并映射到一个新的容器实例, 先更改其仓库配置以全部使用离线仓库:    

```
dash@lucky:~$ sudo mount CentOS-7-x86_64-DVD-1810.iso  /mnt
mount: /mnt: WARNING: device write-protected, mounted read-only.
dash@lucky:~$ sudo docker run -it --name buildavd  -v /mnt:/mnt centos:7.6.1810 /bin/bash
[root@3c49cf47c327 /]# ls /mnt
CentOS_BuildTag  EULA  LiveOS    RPM-GPG-KEY-CentOS-7          TRANS.TBL  isolinux
EFI              GPL   Packages  RPM-GPG-KEY-CentOS-Testing-7  images     repodata
[root@3c49cf47c327 /]# mkdir /etc/yum.repos.d/back
[root@3c49cf47c327 /]# mv /etc/yum.repos.d/* /etc/yum.repos.d/back/
mv: cannot move '/etc/yum.repos.d/back' to a subdirectory of itself, '/etc/yum.repos.d/back/back'
```
拷贝本地仓库定义文件至容器:    

```
# vim local.repo
[LocalRepo]
name=LocalRepository
baseurl=file:///mnt
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
# sudo docker cp local.repo buildavd:/etc/yum.repos.d/
```
进入容器安装必要的包:   

```
[root@3c49cf47c327 /]#  yum groupinstall "X Window System" -y
[root@3c49cf47c327 /]# yum install -y gnome-terminal net-tools java-1.8.0-openjdk tmux celt051 librdkafka
[root@3c49cf47c327 /]# yum -y install vim sudo wget which net-tools bzip2 numpy mailcap firefox
[root@3c49cf47c327 /]# yum -y install xorg-x11-fonts* xulrunner
[root@3c49cf47c327 /]# yum -y groups install "Fonts"
```
使用epel仓库安装`icewm`:    

```
[root@3c49cf47c327 /]# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
[root@3c49cf47c327 /]# yum install -y icewm
[root@3c49cf47c327 /]# yum -y install nss_wrapper gettext
[root@3c49cf47c327 /]# yum erase -y *power* *screensaver*
[root@3c49cf47c327 /]# mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/back/
```

安装openssh相关包:   
```
[root@3c49cf47c327 /]# yum install -y openssh-server openssh-clients
```
从sourceforge下载tigervnc包(`https://sourceforge.net/projects/tigervnc/files/stable/1.10.1/el7/RPMS/`), 这里选择1.10.1版本，以保证参数可以直接传递， 1.11版本后的tigervnc有较大改变，不推荐使用:    

```
[root@3c49cf47c327 tigervnc110]# ls
tigervnc-1.10.1-4.el7.x86_64.rpm        tigervnc-license-1.10.1-4.el7.noarch.rpm  tigervnc-server-minimal-1.10.1-4.el7.x86_64.rpm
tigervnc-icons-1.10.1-4.el7.noarch.rpm  tigervnc-server-1.10.1-4.el7.x86_64.rpm
[root@3c49cf47c327 tigervnc110]# yum install *.rpm
```
安装novnc相关包:    

```
[root@3c49cf47c327 ]# NO_VNC_HOME=/headless/noVNC
[root@3c49cf47c327 ]# mkdir -p $NO_VNC_HOME/utils/websockify
[root@3c49cf47c327 ]# wget -qO- https://github.com/novnc/noVNC/archive/v1.0.0.tar.gz | tar xz --strip 1 -C $NO_VNC_HOME
[root@3c49cf47c327 ]# ls /headless/noVNC/
LICENSE.txt  README.md  app  core  docs  karma.conf.js  package.json  po  tests  utils  vendor  vnc.html  vnc_lite.html
[root@3c49cf47c327 ]# wget -qO- https://github.com/novnc/websockify/archive/v0.6.1.tar.gz | tar xz --strip 1 -C $NO_VNC_HOME/utils/websockify
[root@3c49cf47c327 ]# wget -qO- http://209.141.35.192/v0.6.1.tar.gz | tar xz --strip 1 -C $NO_VNC_HOME/utils/websockify
[root@3c49cf47c327 ]# chmod +x -v $NO_VNC_HOME/utils/*.sh
mode of '/headless/noVNC/utils/launch.sh' retained as 0775 (rwxrwxr-x)
[root@3c49cf47c327 ]# ln -s $NO_VNC_HOME/vnc_lite.html $NO_VNC_HOME/index.html
```
(选做)更换`/usr/local/`目录，这里是因为我们的框架所依赖的库都放在了这个目录下:    

```
[root@3c49cf47c327 ]# mv /usr/local/ /usr/local.back

-------------------------
主机上:   
sudo docker cp /workspace/local/ buildavd:/usr/
然后回到容器中检查
-------------------------

[root@3c49cf47c327 ]# du -hs /usr/local
486M	/usr/local
[root@3c49cf47c327 ]# ls /usr/local
bin  etc  games  include  lib  lib64  libexec  sbin  share  src
```
主机上提交我们对容器的更改并构建中间制品:   

```
$ sudo docker commit buildavd runavd:latest
sha256:ad3c78f39bce2e8ff7492c09cacde9c9f8f5041de6878e92f4423b3d1ba943d4
$ sudo docker images | grep runavd
runavd                           latest                                       ad3c78f39bce   28 seconds ago   2.25GB
```
克隆仓库并构建最终制品:    

```
# git clone  https://github.com/purplepalmdash/runavd.git
# cd runavd
# sudo docker build -t runemu .
# sudo docker images | grep runemu
runemu                           latest                                       1b7b0f283bf0   About a minute ago   2.25GB
```
映射avd镜像目录至容器中并运行:    

```
$ sudo docker run -d --privileged -p 5903:5901 -p 6903:6901 -e VNC_PW=xxxxxx  --user 0  -v /home/dash/Code/android-9:/home/avd runemu:latest
$ sudo docker ps | grep 5903
4fcd1fd2bccc   runemu:latest           "/dockerstartup/vnc_…"   7 minutes ago       Up 7 minutes       0.0.0.0:5903->5901/tcp, :::5903->5901/tcp, 0.0.0.0:6903->6901/tcp, :::6903->6901/tcp   elated_curie
```

### 迁移/运行
导出镜像并传输到别的机器上:    

```
# sudo docker save -o runemu.tar runemu:latest
# scp ./runemu.tar xxx@xxx.xxx.xx.xxx:~
# scp -r /home/dash/Code/android-9/ xxx@xxx.xxx.xxx.xxx:~
在对端机器上:  
$ sudo docker load<runemu.tar
```
运行时icewm失败，需要切换到另个轻量级桌面:   

```
[root@3c49cf47c327 ~]# mv /etc/yum.repos.d/back/epel.repo /etc/yum.repos.d
[root@3c49cf47c327 ~]# yum -y -x gnome-keyring --skip-broken groups install "Xfce"
[root@3c49cf47c327 ~]# mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/back/
```
commit并重新编译:    

```
$ sudo docker commit buildavd runavdxfce:latest
$ sudo docker build -f Dockerfile.centos.xfce.vnc -t runemuxfce4:latest .
```
替换镜像为xfce4的镜像，则可以访问.   

运行镜像:    

```
$ docker run -d --privileged -p 15901:5901 -p 16901:6901 -e VNC_PW=yiersansi --user -0 -v /root/android-9:/home/avd runemuxfce4:latest
```
窗口如下:    

![/images/2022_03_02_16_41_17_855x639.jpg](/images/2022_03_02_16_41_17_855x639.jpg)

在打开的终端里, 开启实例，使能上网:    

```
$ cd /home/avd
$ source env_setup.sh
$ android create avd --name test_liutao_9 --target android-28 --abi x86_64 --device "Nexus 4" --skin 720x1280
$ LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib emulator -avd  test_liutao_9  -verbose -show-kernel -no-snapshot -no-window -cores 4 -memory 4096 -writable-system  -partition-size 65536 -port 5654 -gpu swiftshader_indirect -qemu -cpu host -vnc :50
$ vncviewer localhost:5950
$ adb shell
ip route add default via 192.168.232.1 dev wlan0
ip rule add pref 2 from all lookup default
ip rule add from all lookup main pref 30000
```
结果:   

![/images/2022_03_02_16_44_32_1170x957.jpg](/images/2022_03_02_16_44_32_1170x957.jpg)

