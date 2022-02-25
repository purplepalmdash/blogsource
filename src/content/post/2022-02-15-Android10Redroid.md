+++
title= "Android10Redroid"
date = "2022-02-15T14:10:26+08:00"
description = "Android10Redroid"
keywords = ["Technology"]
categories = ["Technology"]
+++
### aosp preparation
Prepare the `10.0.0_r33` aosp source via:    

```
repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-10.0.0_r33
repo sync -j8
```
Build via:     

```
# vim build/target/product/AndroidProducts.mk
.....
COMMON_LUNCH_CHOICES := \
    aosp_arm64-eng \
    aosp_arm-eng \
    aosp_x86_64-eng \
    aosp_x86-eng \
    sdk_phone_x86_64-userdebug \
# source build/envsetup.sh
# lunch sdk_phone_x86_64-userdebug
# m -j128
```
Then we could use emulator for operating the android 10 vm.    
### Kernel Prepartion
Sync the 4.14.112 kernel via:     

```
git clone https://android.googlesource.com/kernel/goldfish.git
cd goldfish/
git checkout -b android-goldfish-4.14-gchips remotes/origin/android-goldfish-4.14-gchips
vim security/selinux/include/classmap.h 
vim scripts/selinux/mdp/mdp.c 
vim scripts/selinux/genheaders/genheaders.c 
cp arch/x86/configs/x86_64_ranchu_defconfig  arch/x86/configs/x86_64_emu_defconfig
export PATH=$PATH:/root/Code/android10_redroid/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin
export ARCH=x86_64
export CROSS_COMPILE=x86_64-linux-android-
export REAL_CROSS_COMPILE=x86_64-linux-android-
/root/Code/android10_redroid/prebuilts/qemu-kernel/build-kernel.sh --arch=x86_64
cp /tmp/kernel-qemu/x86_64-4.14.112/kernel-qemu  ~/
```
### Kernel Customization
Customize via:    

```
# cd goldfish
# make x86_64_emu_defconfig
# make menuconfig
```
Here you will see the kernel configuration window, make changes in it, then save and replace the `x86_64_emu_defconfig` configuration file.    

![/images/2022_02_15_14_19_00_973x628.jpg](/images/2022_02_15_14_19_00_973x628.jpg)

Detailed changes:    

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
```
But we lost the binderfs support in 4.14.112, have to change to other kernel version!  

Migrating the binderfs to the kernel 4.14.112, steps remains to be written .    

Refers to:   

```
https://github.com/purplepalmdash/binderfs_backport.git
```

Start the emulator via:     

```
# emulator -show-kernel -kernel /root/kernel-qemu -no-snapshot-load -selinux disabled
``` 
Replace the kernel in aosp kernel source:     

```
cd /root/Code/android10_redroid/prebuilts/qemu-kernel/x86_64
cp -r 4.14/ 4.14.back
cp /root/kernel-qemu 4.14/kernel-qemu2 
```
### binderfs enable
aosp source code should add modification for enable binderfs.    
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
Recompile the aosp source code and get the new generated image

### Docker Integration
Download the docker binary files and extract them to prebuilts folder:      

```
$ wget https://download.docker.com/linux/static/stable/x86_64/docker-20.10.8.tgz
// Switch to aosp source tree
$ cd prebuilts
$ tar xzvf ~/docker-20.10.8.tgz -C .
```
Add docker binary files into `system.img`, add them into `/system/bin` so that
we could direct use them:    

```
$ vim  ./build/make/target/board/generic_x86_64/device.mk
// At the end of the file
PRODUCT_COPY_FILES += \
    prebuilts/docker/containerd:system/bin/containerd \
    prebuilts/docker/containerd-shim:system/bin/containerd-shim \
    prebuilts/docker/containerd-shim-runc-v2:system/bin/containerd-shim-runc-v2 \
    prebuilts/docker/ctr:system/bin/ctr \
    prebuilts/docker/docker:system/bin/docker \
    prebuilts/docker/dockerd:system/bin/dockerd \
    prebuilts/docker/docker-init:system/bin/docker-init \
    prebuilts/docker/docker-proxy:system/bin/docker-proxy \
    prebuilts/docker/runc:system/bin/runc \
$ vim build/target/product/sdk_phone_x86_64.mk
// At the end of the file
PRODUCT_ARTIFACT_PATH_REQUIREMENT_ALLOWED_LIST := \
    system/bin/containerd \
    system/bin/containerd-shim \
    system/bin/containerd-shim-runc-v2 \
    system/bin/ctr \
    system/bin/docker \
    system/bin/dockerd \
    system/bin/docker-init \
    system/bin/docker-proxy \
    system/bin/runc \
```
Change the sepolicy for creating the docker runtime:    

```
$ vim system/sepolicy/prebuilts/api/29.0/private/file_contexts

// Added /var,/run,/system/etc/docker definition under # Symlinks
# Symlinks
/bin                u:object_r:rootfs:s0
/bugreports         u:object_r:rootfs:s0
/charger            u:object_r:rootfs:s0
/d                  u:object_r:rootfs:s0
/etc                u:object_r:rootfs:s0
/sdcard             u:object_r:rootfs:s0
/var                u:object_r:rootfs:s0
/run                u:object_r:rootfs:s0
/system/etc/docker                u:object_r:system_file:s0

$ vim system/sepolicy/private/file_contexts
 /sdcard             u:object_r:rootfs:s0
 /var             u:object_r:rootfs:s0
 /run             u:object_r:rootfs:s0
 /system/etc/docker             u:object_r:system_file:s0
 
   # SELinux policy files

$ vim system/core/rootdir/Android.mk

     ln -sf /system/etc $(TARGET_ROOT_OUT)/etc; \
     ln -sf /data/var $(TARGET_ROOT_OUT)/var; \
     ln -sf /data/run $(TARGET_ROOT_OUT)/run; \
     ln -sf /data/user_de/0/com.android.shell/files/bugreports $(TARGET_ROOT_OUT)/bugreports; 


 # Since init.environ.rc is required for init and satisfies that requirement, we hijack it to create the symlink.
 LOCAL_POST_INSTALL_CMD += ; ln -sf /system/bin/init $(TARGET_ROOT_OUT)/init
 LOCAL_POST_INSTALL_CMD += ; ln -sf /data/docker $(TARGET_OUT)/etc/
 LOCAL_POST_INSTALL_CMD += ; ln -sf /data/resolv.conf $(TARGET_OUT)/etc/resolv.conf
```
Manually create the folders and make image again:   

```
$ mkdir -p out/target/product/generic_x86_64/data/run
$ mkdir -p out/target/product/generic_x86_64/data/var
$ mkdir -p out/target/product/generic_x86_64/data/docker
$ echo "nameserver 223.5.5.5" > out/target/product/generic_x86_64/data/resolv.conf
$ make userdataimage -j50
```
Restart the emulator, now it's free to use docker.   

### Create emulator
Start the emulator via:    

```
# sudo tunctl
# brctl addif virbr0 tap0
# ip link set dev tap0 up
# emulator -show-kernel -no-snapshot-load -selinux disabled  -qemu -cpu host -device virtio-net-pci,netdev=hn0,mac=52:55:00:d1:55:51   -netdev tap,id=hn0,ifname=tap0,script=no,downscript=no
```
The added eth1 has no ip addr, use dhclient for getting the address from
virbr0:       

```
adb root
adb shell "dhcpclient -i eth1 &"
```
Check the ip addr for eth1:    

```
adb shell
generic_x86_64:/ # ifconfig eth1
eth1      Link encap:Ethernet  HWaddr 52:55:00:d1:55:51  Driver virtio_net
          inet addr:192.168.122.124  Bcast:192.168.122.255  Mask:255.255.255.0 
          inet6 addr: fe80::5055:ff:fed1:5551/64 Scope: Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:27 errors:0 dropped:0 overruns:0 frame:0 
          TX packets:58 errors:0 dropped:0 overruns:0 carrier:0 
          collisions:0 txqueuelen:1000 
          RX bytes:2800 TX bytes:15341 
```
Set the `/etc/resolv.conf`, cgroupfs, then start the dockerd manually:      

```
echo "nameserver 223.5.5.5">/etc/resolv.conf
mount -t tmpfs -o uid=0,gid=0,mode=0755 cgroup /sys/fs/cgroup
cd /sys/fs/cgroup/
mkdir -p cpu cpu acct blkio memory devices pids
mount -n -t cgroup -o cpu cgroup cpu
mount -n -t cgroup -o  cpuacct cgroup cpuacct
mount -n -t cgroup -o  blkio cgroup blkio
mount -n -t cgroup -o  memory cgroup memory
mount -n -t cgroup -o  devices cgroup devices
mount -n -t cgroup -o  pids cgroup pids

ip rule add from all lookup main pref 30000
dockerd  --dns=223.5.5.5 --data-root=/data/var/ --ip=192.168.122.124 & >/data/dockerd-logfile 2>&1
```
Start the redroid instance via:    

```
docker run -d --privileged -p 8888:5555 redroid/redroid:8.1.0-latest
```
