+++
title= "WorkingTipsonWayDroid"
date = "2021-09-20T12:33:48+08:00"
description = "WorkingTipsonWayDroid"
keywords = ["Technology"]
categories = ["Technology"]
+++
### AIM
Enable `libndk_translation` in waydroid.    

### Steps
Disable the `waydroid-container` and reboot the machine:    

```
systemctl disable waydroid-container
reboot
```
Now you could make a backup for your origin `system.img` and `vendor.img`
file:    

```
 cp /var/lib/waydroid/images/system.img /root 
 cp /var/lib/waydroid/images/vendor.img /root
```
Clone the repository :    

```
$ git clone https://github.com/newbit1/libndk_translation_Module.git
$ cd libndk_translation_Module
$ tar czvf native-bridge.tar.gz system
```
Copy the `native-bridge.tar.gz` to some place(For example /root), later we will use it.     

Resize the img file(enlarge them):    

```
qemu-img resize -f raw system.img +512M
losetup -f system.img
losetup -l
e2fsck -f /dev/loop5 
resize2fs /dev/loop5 
losetup -d /dev/loop5 
 qemu-img resize -f raw vendor.img +100M
 losetup -f vendor.img 
 e2fsck -f /dev/loop6
 resize2fs /dev/loop6 
 losetup -d /dev/loop6 
```

Mount the img file in `rw` mode:    

```
 mount -o rw /var/lib/waydroid/images/system.img /var/lib/waydroid/rootfs
 mount -o rw /var/lib/waydroid/images/vendor.img /var/lib/waydroid/rootfs/vendor
```
Inject `libndk_translation_Module`:    

```
# cd /var/lib/waydroid/rootfs
# cp /root/native-bridge.tar.gz .
# tar xzvf native-bridge.tar.gz
```
Enable `nativebridge.rc`:    

```
# vim /var/lib/waydroid/rootfs/vendor/etc/init/nativebridge.rc
on early-init
    setprop ro.odm.product.cpu.abilist x86_64,x86,arm64-v8a,armeabi-v7a,armeabi
    setprop ro.odm.product.cpu.abilist32 x86,armeabi-v7a,armeabi
    setprop ro.odm.product.cpu.abilist64 x86_64,arm64-v8a
    setprop ro.product.cpu.abilist x86_64,x86,arm64-v8a,armeabi-v7a,armeabi
    setprop ro.product.cpu.abilist32 x86,armeabi-v7a,armeabi
    setprop ro.product.cpu.abilist64 x86_64,arm64-v8a
    setprop ro.vendor.product.cpu.abilist x86_64,x86,arm64-v8a,armeabi-v7a,armeabi
    setprop ro.vendor.product.cpu.abilist32 x86,armeabi-v7a,armeabi
    setprop ro.vendor.product.cpu.abilist64 x86_64,arm64-v8a
    setprop ro.dalvik.vm.native.bridge libndk_translation.so
    setprop ro.enable.native.bridge.exec 1
    setprop ro.ndk_translation.version 0.2.2
    setprop ro.dalvik.vm.isa.arm x86
    setprop ro.dalvik.vm.isa.arm64 x86_64
```
Enable `native.bridge` in `prop.default`:    

```
# vim /var/lib/waydroid/rootfs/system/etc/prop.default
native.bridge=0 --> native.bridge=1
?
native.bridge=0 --> native.bridge=libndk_translation.so
```
trancode not OK...... lots of issues.  
