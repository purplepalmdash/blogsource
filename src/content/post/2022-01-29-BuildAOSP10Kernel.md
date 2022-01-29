+++
title= "BuildAOSP10Kernel"
date = "2022-01-29T19:36:32+08:00"
description = "BuildAOSP10Kernel"
keywords = ["Technology"]
categories = ["Technology"]
+++
为模拟器emulator编译aosp10的内核.    

### 内核源码获取
需要获取模拟器自身的内核代码，方法是通过`adb shell`进入到shell下运行`uname -a`。    

尽量找和默认内核差不多的版本的内核进行编译，否则有可能编译出的内核无法启动系统。这里选择的是`4.14.112+`.    

```
$ git clone https://android.googlesource.com/kernel/goldfish.git
$ git checkout -b android-goldfish-4.14-gchips remotes/origin/android-goldfish-4.14-gchips
$ git branch 
* android-goldfish-4.14-gchips
  master
```
获取的内核源码架构如下：    

```
# ls
arch                          build-kernel.sh  CREDITS        firmware  ipc      lib          modules.builtin  README    sound       verity_dev_keys.x509
block                         built-in.o       crypto         fs        Kbuild   MAINTAINERS  modules.order    samples   System.map  virt
build.config.goldfish.arm64   certs            Documentation  include   Kconfig  Makefile     Module.symvers   scripts   tools       vmlinux
build.config.goldfish.x86_64  COPYING          drivers        init      kernel   mm           net              security  usr         vmlinux.o
```
### 内核编译链获取
内核编译链在编译好的`aosp10`源码下，编译方法不再详细讲解。    

```
# ls prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/
bin  COPYING  COPYING3  COPYING3.LIB  COPYING.LIB  COPYING.RUNTIME  lib  libexec  MODULE_LICENSE_GPL  NOTICE  OWNERS  repo.prop  x86_64-linux-android
# ls prebuilts/qemu-kernel/build-kernel.sh
prebuilts/qemu-kernel/build-kernel.sh
```
### 编译
`android-goldfish-4.14-gchips`内核中需要做以下修改以通过编译:     

```
# vim /security/selinux/include/classmap.h
添加:
    #include <linux/socket.h>
# vim scripts/selinux/mdp/mdp.c 
# scripts/selinux/genheaders/genheaders.c
去掉头文件包含的 #include <sys/socket.h>
```
编译配置:    

```
export PATH=$PATH:/home/xxxxx/Code/aosp10/prebuilts/gcc/linux-x86/x86/x86_64-linux-android-4.9/bin
export ARCH=x86_64
export CROSS_COMPILE=x86_64-linux-android-
export REAL_CROSS_COMPILE=x86_64-linux-android-
```
创建内核定义文件：     

```
# cp arch/x86/configs/x86_64_ranchu_defconfig  arch/x86/configs/x86_64_emu_defconfig
```
编译:   

```
# /home/xxxx/Code/aosp10/prebuilts/qemu-kernel/build-kernel.sh --arch=x86_64
```
最终编译成品:    

```
# cp /tmp/kernel-qemu/x86_64-4.14.112/kernel-qemu  ~/
```
### 测试内核
emulator启动的时候显式指定所需的内核:    

```
# emulator -show-kernel -kernel ~/kernel-qemu
```
检测内核的方式同样是通过`adb shell`进入到命令行界面下查看内核。    
