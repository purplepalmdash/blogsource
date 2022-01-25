+++
title= "RunWaydroidUbportsOnPixel3a"
date = "2022-01-25T08:00:02+08:00"
description = "RunWaydroidUbportsOnPixel3a"
keywords = ["Technology"]
categories = ["Technology"]
+++
最近会研究一些安卓虚拟化的知识，这里记录下来在Pixel
3a上刷Ubports后，在Ubports中启用waydroid的过程。    

### 1. 准备工作/相关概念
Pixel
3a一支，淘宝很容易买到，注意要买可解锁BootLoader的，价格大概是四五百块，取决于成色。    

选择Pixel
3a是因为它是Ubports官方支持中比较完善的几个之一，官方的设备支持列表如下：    
[https://devices.ubuntu-touch.io/?pk_vid=5294f9f7280f826e164306913156a7ee](https://devices.ubuntu-touch.io/?pk_vid=5294f9f7280f826e164306913156a7ee)     

当然如果手头有别的可以刷的手机也可以尝试下, Pixel
3a的刷机体验是比较好的。   

Ubports: Ubuntu touch.    
Waydroid: 以前称为Anbox-Halium，是Anbox 的重建版本，旨在使用比Anbox
更新更流畅的安卓算力体验。

### 2. 操作指南
#### 2.1 刷入安卓9
参考[https://devices.ubuntu-touch.io/device/sargo/](https://devices.ubuntu-touch.io/device/sargo/)   

全翻墙的情况下，访问:    

[https://flash.android.com/release](https://flash.android.com/release),
按提示操作, 记得选择图示为`PQ3B.190801.002`的安卓9版本。

![/images/2022_01_25_09_06_50_621x486.jpg](/images/2022_01_25_09_06_50_621x486.jpg)

看到此提示则代表刷入已经成功:   

![/images/2022_01_25_07_58_49_612x183.jpg](/images/2022_01_25_07_58_49_612x183.jpg)
刷入成功后会提示需要lock bootloader，
按照提示先将bootloader锁定后，进入到安卓9中执行初始化操作.
接着进入到解锁bootloader以便刷Ubports.    

#### 2.2 解锁bootloader:  
首先要做的是开启Usb调试
`关于手机`->`版本号`上狂点，直至打开`开发者模式`，
而后`系统`->`开发者选项`->`调试`->`USB调试`.     

```
# adb devices
List of devices attached
xxxxxx5	device

# adb reboot bootloader
```
此时手机会停在fastboot阶段，在终端中继续敲入

```
# fastboot flashing unlock
OKAY [  0.338s]
Finished. Total time: 0.338s
```
此时在手机上按音量-键选择到`unlock
bootloader`的选项后，继续按电源键确定。bootloader解锁成功后需重新启动并重新设置手机。此时需要重新开启usb调试等。   


#### 2.3 刷入ubports
再次强调: ubports刷入的先决条件是pixel 3a上运行的安卓版本为`PQ3B.190801.002`。

ArchLinux上通过snap安装ubports刷机软件, 参考网址如下:     

[https://snapcraft.io/install/ubports-installer/arch](https://snapcraft.io/install/ubports-installer/arch)

开启刷机软件开始刷ubports:    

```
sudo ubports-installer
```

![/images/2022_01_25_08_25_47_803x441.jpg](/images/2022_01_25_08_25_47_803x441.jpg)

确认手机型号:    

![/images/2022_01_25_08_26_04_657x371.jpg](/images/2022_01_25_08_26_04_657x371.jpg)

选择devel版(如果不是为了体验waydroid也可以选择为stable版), 一定要选择`wipe userdata`，否则刷机将一直卡顿不会成功:   

![/images/2022_01_25_08_26_38_621x462.jpg](/images/2022_01_25_08_26_38_621x462.jpg)

这里提示在fastboot菜单中需要选择为`Recovery
Mode`才可以继续安装，在手机上按音量-键结合电源键选择，进入到恢复模式后点下一步:   

![/images/2022_01_25_08_27_53_716x365.jpg](/images/2022_01_25_08_27_53_716x365.jpg)

现在`ubports-installer`将自动下载镜像并进行刷机，刷机速度取决于下载速度,
推荐在空闲时段操作，在早上7、8点时大概几分钟时间刷好:    

![/images/2022_01_25_08_29_20_554x581.jpg](/images/2022_01_25_08_29_20_554x581.jpg)

刷机软件提示出以下界面的时候，手机将重启，这时需要耐心等待手机上的刷机进程完成写入操作，刷机过程中会有个黄色的小标志在不停转动， 而后将重启:    

![/images/2022_01_25_08_31_21_651x492.jpg](/images/2022_01_25_08_31_21_651x492.jpg)

经过一系列设置之后，Ubports将启动成功并进入到系统中。   

#### 2.4 安装waydroid
确认自己安装的是开发者版本，如果是stable的话，在`关于`->`检查更新`下的配置按钮中，切换成开发者版本就可以了:    

![/images/2022_01_25_08_47_25_478x309.jpg](/images/2022_01_25_08_47_25_478x309.jpg)

`waydroid`中也是可以使用adb命令的，使用adb命令连接到手机上后，运行waydroid的安装操作,
adb命令的一个参考如下(以下参考演示了如何连接到手机、如何从手机取回截屏):     

```
$ adb devices
List of devices attached
92UAY04L95	device

$ adb shell
phablet@ubuntu-phablet:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  Videos
phablet@ubuntu-phablet:~$ cd Pictures/
phablet@ubuntu-phablet:~/Pictures$ ls
Screenshots
phablet@ubuntu-phablet:~/Pictures$ cd Screenshots/
phablet@ubuntu-phablet:~/Pictures/Screenshots$ ls
screenshot20220125_084411555.png
phablet@ubuntu-phablet:~/Pictures/Screenshots$ exit

$ adb pull /home/phablet/Pictures/Screenshots/screenshot20220125_084411555.png
/home/phablet/Pictures/Screenshots/screenshot20220125_084411555.png: 1 file pulled, 0 skipped. 2.7 MB/s (76661 bytes in 0.027s)
```
可以检查一下磁盘占用情况, 确保安装waydroid具备足够的磁盘空间:    

```
dfphablet@ubuntu-phablet:~$ df -h
Filesystem       Size  Used Avail Use% Mounted on
/dev/root        3.0G  2.2G  804M  73% /
devtmpfs         1.7G  604K  1.7G   1% /dev
tmpfs            1.8G  1.3M  1.8G   1% /run
/dev/loop0       268M  267M     0 100% /android
/dev/mmcblk0p72   49G   27M   47G   1% /userdata
none             4.0K     0  4.0K   0% /sys/fs/cgroup
cgmfs            100K     0  100K   0% /run/cgmanager/fs
tmpfs            1.8G   32K  1.8G   1% /tmp
none             5.0M     0  5.0M   0% /run/lock
none             1.8G  172K  1.8G   1% /run/shm
none             100M     0  100M   0% /run/user
tmpfs            1.8G     0  1.8G   0% /media
tmpfs            1.8G     0  1.8G   0% /var/lib/openvpn/chroot/tmp
tmpfs            1.8G     0  1.8G   0% /var/lib/sudo
/dev/mmcblk0p71  755M  512M  228M  70% /android/vendor
tmpfs            1.8G     0  1.8G   0% /mnt/vendor
/dev/mmcblk0p48   35M  3.8M   30M  12% /mnt/vendor/persist
/dev/mmcblk0p66   12M   80K   11M   1% /android/metadata
/dev/mmcblk0p22   80M   72M  8.8M  90% /android/vendor/firmware_mnt
tmpfs            356M   40K  356M   1% /run/user/32011
tmpfs            356M     0  356M   0% /run/user/0
```
`adb shell`下输入如下命令安装`waydroid`:    

```
sudo -s
sudo mount -o remount,rw /
apt update
apt install waydroid -y
waydroid init
```
值得注意的是，`waydroid
init`需要从sourceforge拉取700MB左右的镜像，由于众所周知的原因，国内网络一直不是很好，这一步可能会耗费很长的时间。个人建议是在网络空闲时段(例如起个大早来跑)运行该操作。当然具备动手能力的可以用全翻墙网路来跑，速度会快很多。    

waydroid命令实际上是一个python脚本，理论上是可以改一下调整它的镜像初始化机制以使用本地包用于离线安装的, 有兴趣的可以更改源码。   

```
root@ubuntu-phablet:~# which waydroid
/usr/bin/waydroid
root@ubuntu-phablet:~# cd /usr/lib/waydroid/
root@ubuntu-phablet:~# grep -i "http" ./ -r | grep channel
./tools/config/__init__.py:    "system_channel": "http://ota.waydro.id/system",
./tools/config/__init__.py:    "vendor_channel": "http://ota.waydro.id/vendor",
```
waydroid初始化过程:    

```
root@ubuntu-phablet:~# waydroid init
[08:55:02] Download https://sourceforge.net/projects/waydroid/files/images/system/lineage/waydroid_arm64/lineage-17.1-20211021-VANILLA-waydroid_arm64-system.zip/download
[09:04:37] Validating system image
[09:04:38] Extracting to /var/lib/waydroid/images
[09:05:22] Download https://sourceforge.net/projects/waydroid/files/images/vendor/waydroid_arm64/lineage-16.0-20211020-HALIUM_9-waydroid_arm64-vendor.zip/download
[09:05:42] Validating vendor image
[09:05:42] Extracting to /var/lib/waydroid/images
```
此时需要重启手机以继续安装。     

####  2.5 启动waydroid
启动完毕后，首先启动waydroid容器:   

```
$ adb shell
phablet@ubuntu-phablet:~$ sudo waydroid container start
[sudo] password for phablet: 

```
然后在另一个adb shell中启动waydroid session，
此时第一个shell里的命令会退出，接着就可以点开waydroid了:    

```
$ waydroid session start
[09:15:30] XDG Session is not "wayland"
[09:15:32] Failed to start Clipboard manager service, check logs
[09:15:51] Android with user 0 is ready
```
接着在手机里点开`wayroid`则可以开启waydroid界面.    

启动这一步一直没太搞明白应该怎样才是正确的操作。但经历以上操作后，再次关机重启后，waydroid都可以直接在ubports下直接点开

如果遇到初始化问题不成功需要重新开始的话，可以使用以下命令清零waydroid安装:    

```
apt remove waydroid
apt purge waydroid
sudo rm -rf /var/lib/waydroid /home/.waydroid ~/waydroid ~/.share/waydroid ~/.local/share/applications/*aydroid* ~/.local/share/waydroid
```

#### 2.6 体验waydroid
安装应用宝:    

```
$ adb push MobileAssistant_1.apk /home/phablet/
MobileAssistant_1.apk: 1 file pushed, 0 skipped. 6.1 MB/s (10925622 bytes in 1.720s)
$ apks adb shell
phablet@ubuntu-phablet:~$ sudo waydroid app install ~/MobileAssistant_1.apk
[sudo] password for phablet: 
```

![/images/2022_01_25_09_25_18_406x838.jpg](/images/2022_01_25_09_25_18_406x838.jpg)
接下来的操作就是常规的安装apk, 使用apk。 

鲁大师跑快40万分:    

![/images/2022_01_25_09_49_20_405x834.jpg](/images/2022_01_25_09_49_20_405x834.jpg)

