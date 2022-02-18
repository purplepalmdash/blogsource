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
### Docker Integration
Download the docker binary files and extract them to prebuilts folder:      

```
$ wget https://download.docker.com/linux/static/stable/x86_64/docker-20.10.8.tgz
// Switch to aosp source tree
$ cd prebuilts
$ tar xzvf ~/docker-20.10.8.tgz -C .
```

