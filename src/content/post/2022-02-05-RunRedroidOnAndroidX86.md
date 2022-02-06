+++
title= "RunRedroidOnAndroidX86"
date = "2022-02-05T09:05:02+08:00"
description = "RunRedroidOnAndroidX86"
keywords = ["Technology"]
categories = ["Technology"]
+++
This guideline shows how to run redroid on Android-x86.    
### Introduction
Redroid:    
> ReDroid (Remote anDroid) is a GPU accelerated AIC (Android In Container) solution. You can boot many instances in Linux host (Docker, podman, k8s etc.). ReDroid supports both arm64 and amd64 architectures. ReDroid is suitable for Cloud Gaming, VMI (Virtual Mobile Infrastructure), Automation Test and more
[https://github.com/remote-android/redroid-doc#overview](https://github.com/remote-android/redroid-doc#overview)

Android-x86:    
> Run Android on your PC. This is a project to port Android Open Source Project to x86 platform, formerly known as "patch hosting for android x86 support".   
[https://www.android-x86.org/](https://www.android-x86.org/)   

### Building Android-x86
We use `pie`(based on aosp9) for startup:    

```
# mkdir android-x86
# cd android-x86
# repo init -u git://git.osdn.net/gitroot/android-x86/manifest -b pie-x86
# repo sync --no-tags --no-clone-bundle -j18
```
Or:    

```
repo init -u git://git.osdn.net/gitroot/android-x86/manifest -b pie-x86 -m android-x86-9.0-r2.xml
repo sync --no-tags --no-clone-bundle
```

repo size:    

```
# du -hs android-x86/
79G	android-x86/
# ls android-x86/
Android.bp  bionic    bootstrap.bash  compatibility  dalvik       device    frameworks  kernel   libnativehelper  packages  platform_testing  sdk     test
art         bootable  build           cts            development  external  hardware    libcore  Makefile         pdk       prebuilts         system  tools
```
Make modification for rootdir, the aim is for enable binderfs:     

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

Build via:    

```
# . build/envsetup.sh; lunch android_x86_64-userdebug
# make iso_img TARGET_PRODUCT=android_x86_64 -j128
```
Examine the build out iso files:    

```
# ls -l -h out/target/product/x86_64/android_x86_64.iso
-rw-r--r-- 1 root root 725M Feb  5 02:02 out/target/product/x86_64/android_x86_64.iso
```
### Customization kernel
customize the kernel via(See last section kernel configuration):    

```
# cd kernel
# make android-x86_64_defconfig
# make menuconfig
##### After make customization
# cp arch/x86/configs/android-x86_64_defconfig arch/x86/configs/android-x86_64_defconfig.back
# cp .config arch/x86/configs/android-x86_64_defconfig 
```
Re-build the kernel and re-gen iso:    

```
# cd kernel
# make mrproper
# cd ..
# . build/envsetup.sh; lunch android_x86_64-userdebug
# rm $OUT/obj/kernel/arch/x86/boot/bzImage
# make iso_img TARGET_PRODUCT=android_x86_64 -j128
```
### Android-x86 Installation
Install Android-x86 in vm:    

![/images/2022_02_05_16_38_45_562x474.jpg](/images/2022_02_05_16_38_45_562x474.jpg)

Choose Partition:    

![/images/2022_02_05_16_39_12_661x200.jpg](/images/2022_02_05_16_39_12_661x200.jpg)

Choose GPT or not:     

![/images/2022_02_05_16_56_29_367x174.jpg](/images/2022_02_05_16_56_29_367x174.jpg)   

cfdisk:    

![/images/2022_02_05_17_00_19_740x423.jpg](/images/2022_02_05_17_00_19_740x423.jpg)

create 200 GB primary disk:    

![/images/2022_02_05_17_03_04_714x415.jpg](/images/2022_02_05_17_03_04_714x415.jpg)

After write and quit, Choose partition:    

![/images/2022_02_05_17_04_37_673x346.jpg](/images/2022_02_05_17_04_37_673x346.jpg)

Format to ext4 format:    

![/images/2022_02_05_17_05_03_534x174.jpg](/images/2022_02_05_17_05_03_534x174.jpg)

Confirm:   

![/images/2022_02_05_17_05_56_535x166.jpg](/images/2022_02_05_17_05_56_535x166.jpg)

Formatting:    

![/images/2022_02_05_17_06_17_642x152.jpg](/images/2022_02_05_17_06_17_642x152.jpg)

Confirm:    

![/images/2022_02_05_17_06_39_436x131.jpg](/images/2022_02_05_17_06_39_436x131.jpg)

Read/Write:    

![/images/2022_02_05_17_07_09_554x170.jpg](/images/2022_02_05_17_07_09_554x170.jpg)

Installing status:   

![/images/2022_02_05_17_07_19_642x156.jpg](/images/2022_02_05_17_07_19_642x156.jpg)

Choose Reboot:    

![/images/2022_02_05_17_08_10_480x191.jpg](/images/2022_02_05_17_08_10_480x191.jpg)

### Enable docker
Enable wifi:   

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
Create folder structure:   

```
# adb shell
x86_64:/ # mkdir /data/var                                                                                                                                 
x86_64:/ # mkdir /data/docker/
x86_64:/ # 
```
Download docker-ce binary files from `https://`, extract them and upload them
to android-x86 machine:    

```
# adb push ~/Code/docker/* /data/docker/
```
Prepare the dockerd running environment in `adb shell`:    

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
Start the dockerd daemon:    

```
# dockerd  --dns=223.5.5.5 --data-root=/data/var/ --ip=192.168.122.232 & >/data/dockerd-logfile 2>&1
# docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```
Start redroid docker instance:    

```
# docker run -d --privileged -p 8888:5555 redroid/redroid:9.0.0-latest
# docker ps
CONTAINER ID   IMAGE                          COMMAND                  CREATED         STATUS         PORTS                            NAMES
2efa5f6e987b   redroid/redroid:9.0.0-latest   "/init qemu=1 androi…"   5 seconds ago   Up 4 seconds   192.168.122.232:8888->5555/tcp   sweet_ishizaka
```
### Verification
Connect and view screen via:    

```
# adb connect 192.168.122.232:8888
# scrcpy --serial 192.168.122.232:8888
```

![/images/2022_02_06_09_34_35_427x768.jpg](/images/2022_02_06_09_34_35_427x768.jpg)

Install apks via:    

```
$ adb -s 192.168.122.232:8888 install ~/apks/aaaaaaa.apk
```

### Kernel configuration
Configuration items(based on `android-x86_64_defconfig`):    

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
