+++
title= "RunRedroidOnAndroidX86CN"
date = "2022-02-07T09:05:02+08:00"
description = "RunRedroidOnAndroidX86CN"
keywords = ["Technology"]
categories = ["Technology"]
+++
这篇指南将详细讲述如何在Android-x86上运行redroid容器。
### 介绍
Redroid:    
> ReDroid (Remote anDroid) 是一种支持GPU加速的(容器中的Android)方案. 你可以在任意Linux主机(Docker, podman, k8s等)上使用。Redoird支持arm64及amd64架构，Redroid适用于云游， VMI(Virtual Mobile Infrstructure), 自动化测试及更多场景。    
[https://github.com/remote-android/redroid-doc#overview](https://github.com/remote-android/redroid-doc#overview)

Android-x86:    
> 在PC上运行Android. 这个项目被用于将Android Open Source Project移植到x86平台上，项目前身为"patch hosting for android x86 support".   
[https://www.android-x86.org/](https://www.android-x86.org/)   

### 编译 Android-x86
我们从`pie`(aosp9)分支作为起点:    

```
# mkdir android-x86
# cd android-x86
# repo init -u git://git.osdn.net/gitroot/android-x86/manifest -b pie-x86
# repo sync --no-tags --no-clone-bundle -j18
```
或者，进一步指定其子版本(非master分支):    

```
repo init -u git://git.osdn.net/gitroot/android-x86/manifest -b pie-x86 -m android-x86-9.0-r2.xml
repo sync --no-tags --no-clone-bundle
```

repo 大小为:    

```
# du -hs android-x86/
79G	android-x86/
# ls android-x86/
Android.bp  bionic    bootstrap.bash  compatibility  dalvik       device    frameworks  kernel   libnativehelper  packages  platform_testing  sdk     test
art         bootable  build           cts            development  external  hardware    libcore  Makefile         pdk       prebuilts         system  tools
```
对rootdir作修改，目的是为了启用binderfs:    

```
# vim ./system/core/rootdir/init.rc
    mount configfs none /config nodev noexec nosuid
    chmod 0770 /config/sdcardfs
    chown system package_info /config/sdcardfs

+    # Mount binderfs
+    mkdir /dev/binderfs
+    mount binder binder /dev/binderfs stats=global
+    chmod 0755 /dev/binderfs
+ 
+    # Mount fusectl
+    mount fusectl none /sys/fs/fuse/connections
+ 
+    symlink /dev/binderfs/binder /dev/binder                                            
+    symlink /dev/binderfs/hwbinder /dev/hwbinder
+    symlink /dev/binderfs/vndbinder /dev/vndbinder
+ 
+    chmod 0666 /dev/binderfs/hwbinder
+    chmod 0666 /dev/binderfs/binder
+    chmod 0666 /dev/binderfs/vndbinder
```

使用以下命令编译出iso文件:    

```
# . build/envsetup.sh; lunch android_x86_64-userdebug
# make iso_img TARGET_PRODUCT=android_x86_64 -j128
```
检查编译出的Iso文件:    

```
# ls -l -h out/target/product/x86_64/android_x86_64.iso
-rw-r--r-- 1 root root 725M Feb  5 02:02 out/target/product/x86_64/android_x86_64.iso
```
### 自定义内核
自定义内核步骤如下(详细步骤见最后章节kernel configuration)：    

```
# cd kernel
# make android-x86_64_defconfig
# make menuconfig
##### After make customization
# cp arch/x86/configs/android-x86_64_defconfig arch/x86/configs/android-x86_64_defconfig.back
# cp .config arch/x86/configs/android-x86_64_defconfig 
```
重新编译内核并生成iso文件:    

```
# cd kernel
# make mrproper
# cd ..
# . build/envsetup.sh; lunch android_x86_64-userdebug
# rm $OUT/obj/kernel/arch/x86/boot/bzImage
# make iso_img TARGET_PRODUCT=android_x86_64 -j128
```
### Android-x86 安装
在虚拟机中安装Android-x86:    

![/images/2022_02_05_16_38_45_562x474.jpg](/images/2022_02_05_16_38_45_562x474.jpg)

选择分区:    

![/images/2022_02_05_16_39_12_661x200.jpg](/images/2022_02_05_16_39_12_661x200.jpg)

是否选择GPT:   

![/images/2022_02_05_16_56_29_367x174.jpg](/images/2022_02_05_16_56_29_367x174.jpg)   

cfdisk:    

![/images/2022_02_05_17_00_19_740x423.jpg](/images/2022_02_05_17_00_19_740x423.jpg)

创建一个200GB大小的主分区:    

![/images/2022_02_05_17_03_04_714x415.jpg](/images/2022_02_05_17_03_04_714x415.jpg)

选择Write, Quit后，在安装界面中选择分区进行安装:    

![/images/2022_02_05_17_04_37_673x346.jpg](/images/2022_02_05_17_04_37_673x346.jpg)

选择格式化为ext4格式:    

![/images/2022_02_05_17_05_03_534x174.jpg](/images/2022_02_05_17_05_03_534x174.jpg)

确认:   

![/images/2022_02_05_17_05_56_535x166.jpg](/images/2022_02_05_17_05_56_535x166.jpg)

继续格式化:    

![/images/2022_02_05_17_06_17_642x152.jpg](/images/2022_02_05_17_06_17_642x152.jpg)

确认:    

![/images/2022_02_05_17_06_39_436x131.jpg](/images/2022_02_05_17_06_39_436x131.jpg)

设置为可读/可写模式:    

![/images/2022_02_05_17_07_09_554x170.jpg](/images/2022_02_05_17_07_09_554x170.jpg)

开始安装:   

![/images/2022_02_05_17_07_19_642x156.jpg](/images/2022_02_05_17_07_19_642x156.jpg)

选择 Reboot, 完成安装:    

![/images/2022_02_05_17_08_10_480x191.jpg](/images/2022_02_05_17_08_10_480x191.jpg)

### 使能 docker
在Android-x86界面中，使能 wifi:   

![/images/2022_02_05_18_16_24_926x458.jpg](/images/2022_02_05_18_16_24_926x458.jpg)

adb root and re-connect:    

```
# adb connect 192.168.122.232
connected to 192.168.122.232:5555
# adb -s 192.168.122.232:5555 root
restarting adbd as root
# adb connect 192.168.122.232
connected to 192.168.122.232:5555
```
创建docker工作目录:   

```
# adb shell
x86_64:/ # mkdir /data/var                                                                                                                                 
x86_64:/ # mkdir /data/docker/
x86_64:/ # 
```
从docker-ce官方网站下载docker-ce二进制包， 将其本地解压后通过adb push命令上传到对应目录:    

```
# adb push ~/Code/docker/* /data/docker/
```
在`adb shell`命令行下准备dockerd工作环境:    

```
mount -o remount,rw /
ln -s /data/docker/* /bin/
ip rule add pref 1 from all lookup main
ip rule add pref 2 from all lookup default
ip route add default via 192.168.122.1 dev wlan0
ip rule add from all lookup main pref 30000
echo "nameserver 223.5.5.5">/etc/resolv.conf
# mount cgroupfs
mount -t tmpfs -o uid=0,gid=0,mode=0755 cgroup /sys/fs/cgroup
cd /sys/fs/cgroup/
mkdir -p cpu cpu acct blkio memory devices pids
mount -n -t cgroup -o cpu cgroup cpu
mount -n -t cgroup -o  cpuacct cgroup cpuacct
mount -n -t cgroup -o  blkio cgroup blkio
mount -n -t cgroup -o  memory cgroup memory
mount -n -t cgroup -o  devices cgroup devices
mount -n -t cgroup -o  pids cgroup pids
```
使用以下命令启动dockerd守护进程:   

```
# dockerd  --dns=223.5.5.5 --data-root=/data/var/ --ip=192.168.122.232 & >/data/dockerd-logfile 2>&1
# docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```
此时可以启动redroid容器实例:    

```
# docker run -d --privileged -p 8888:5555 redroid/redroid:9.0.0-latest
# docker ps
CONTAINER ID   IMAGE                          COMMAND                  CREATED         STATUS         PORTS                            NAMES
2efa5f6e987b   redroid/redroid:9.0.0-latest   "/init qemu=1 androi…"   5 seconds ago   Up 4 seconds   192.168.122.232:8888->5555/tcp   sweet_ishizaka
```
### 验证
adb connect并使用scrcpy命令验证redroid实例是否运行正常:    

```
# adb connect 192.168.122.232:8888
# scrcpy --serial 192.168.122.232:8888
```

![/images/2022_02_06_09_34_35_427x768.jpg](/images/2022_02_06_09_34_35_427x768.jpg)

通过以下命令在redroid实例中安装apk:    

```
$ adb -s 192.168.122.232:8888 install ~/apks/aaaaaaa.apk
```

### 内核配置
需追加的内核配置项(基于arch/x86/configs下的 `android-x86_64_defconfig`):    

```
Generic setup -> POSIX Message Queues
Generic setup -> Controller Group support -> PIDs controller
Generic setup -> Controller Group support -> Device controller
Generic setup -> Controller Group support -> CPU controller -> Group sheduling for SCHED_OTHER
Generic setup -> Controller Group support -> CPU controller -> CPU bandwidth provisioning for FAIR-GROUP_SCHED
Generic setup -> Controller Group support -> CPU controller -> Group sheduling for SCHED_RR/FIFO
Generic setup -> Controller Group support -> Perf controller
Generic setup -> Namespaces support -> User namespace
Generic setup -> Namespaces support -> PID namespace
Networking support -> Networking options -> Network packet filtering framework (Netfilter) -> Bridged IP/ARP packets fiiltering
Networking support -> Networking options -> Network packet filtering framework (Netfilter) -> IP virtual server support
Networking support -> Networking options -> Network packet filtering framework (Netfilter) -> Core Netfilter configuration ->  "addrtype" address type match support
Networking support -> Networking options -> Network packet filtering framework (Netfilter) -> Core Netfilter configuration ->  "control group" address type match support
Networking support -> Networking options -> Network packet filtering framework (Netfilter) -> Core Netfilter configuration ->  "control group" address type match support
File Systems -> Overlay filesystem support
Device Driver -> Android ->  Android Binderfs filesystem
```
