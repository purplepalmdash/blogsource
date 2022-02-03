+++
title= "CellsOnAndroid10IntegrationCN"
date = "2022-02-02T23:50:52+08:00"
description = "WorkingTipsOnCellsOnAndroid10"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. cells源码准备
从github克隆代码至本地目录

```
# mkdir -p ~/Code/show/cells
# cd ~/Code/show/cells
# git clone https://github.com/jianglin-code/cells-android10.git
# cd cells-android10
# ls
cells  frameworks  kernel  README.md  system
```

### 2. aosp 10源码准备
对标cells源码中的要求, 克隆特定版本的aosp10源码(`10.0.0_r33`):    

```
# mkdir -p ~/Code/show/aosp10
# cd ~/Code/show/aosp10
# repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-10.0.0_r33
# repo sync -j8
```

### 3. 内核源码准备
因为使用的真机是pixel 3a，内核源码的获取见以下代码:    

```
# mkdir -p ~/Code/show/android-kernel
# cd ~/Code/show/android-kernel
#  repo init --depth 1 -u https://aosp.tuna.tsinghua.edu.cn/kernel/manifest -b android-msm-bonito-4.9-android10
# repo sync -j4
# ls
build  build.config  prebuilts  prebuilts-master  private
# du -hs *
732K	build
0	build.config
189M	prebuilts
5.8G	prebuilts-master
870M	private
```
### 4. 二进制驱动准备
从以下网址查找真机对应需要的二进制驱动标号:   
`https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds`:    

```
QQ2A.200305.002	android-10.0.0_r30	Android10	Pixel 2、Pixel 2 XL、Pixel 3、Pixel 3 XL、Pixel 3a、Pixel 3a XL	2020-03-05
```
根据查找出的代号，从以下网址下载二进制驱动文件:    
`https://developers.google.cn/android/drivers?hl=zh-cn#sargoqq2a.200405.005`:    

![/images/2022_02_02_22_55_30_880x211.jpg](/images/2022_02_02_22_55_30_880x211.jpg)

将下载的二进制驱动文件拷贝到aosp源码树:    

```
# cp extract-google_devices-sargo.sh extract-qcom-sargo.sh /root/Code/show/aosp10
```

### 5. 编译内核
使用`cells`源码中的相应目录替换aosp10内核中对应代码:    

```
# mv private/msm-google private/msm-google.back
# cp -ar  ~/Code/show/cells/cells-android10/kernel/ private/msm-google
```
使用以下命令编译内核:    

```
# cd private/msm-google
# make mrproper
# cd ../../
# cp -r private/msm-google.back/techpack/ private/msm-google/
# vim build/build.sh
comment the line of soong_zip(line 798)
# build/build.sh
```
检验内核编译出的二进制文件:    

```
# ls out/android-msm-pixel-4.9/dist/
abi.prop  Image.lz4              kernel-uapi-headers.tar.gz  System.map  wlan.ko
dtbo.img  kernel-headers.tar.gz  sdm670.dtb                  vmlinux
```
### 6. aosp源码编译
解压二进制驱动代码:    

```
# ./extract-google_devices-sargo.sh 
# ./extract-qcom-sargo.sh 
```
替换aosp中内置的内核代码:   

```
# mv device/google/bonito-kernel/Image.lz4 device/google/bonito-kernel/Image.lz4.back
# cp /root/Code/show/android-kernel/out/android-msm-pixel-4.9/dist/Image.lz4 device/google/bonito-kernel/Image.lz4
```
将cells源码并入到aosp编译树中:    

```
# cp -r /root/Code/show/cells/cells-android10/cells/ vendor/
# vim device/google/bonito/device.mk
Added at the last line:   
$(call inherit-product-if-exists, vendor/cells/cells_build.mk)
```
开始编译:    

```
# source build/envsetup.sh
# lunch aosp_sargo-userdebug
# vim frameworks/base/data/etc/privapp-permissions-platform.xml
增加: 
    <privapp-permissions package="com.cells.cellswitch.secure">
        <permission name="android.permission.BLUETOOTH_PRIVILEGED"/>
    </privapp-permissions>
# vim vendor/cells/switchsystem/src/com/cells/cellswitch/secure/view/SwitchActivity.java
注释掉: 
//import android.os.CellsManager;
//import android.os.ICellsManager;
# cp -r /root/Code/show/aosp10back/frameworks/multidex/library/
frameworks/multidex
# m -j128
```
编译到80%左右时将失败，重新计算依赖后编译方可成功:    

```
# development/vndk/tools/header-checker/utils/create_reference_dumps.py  -l libbinder && development/vndk/tools/header-checker/utils/create_reference_dumps.py  -l libhwbinder_noltopgo && development/vndk/tools/header-checker/utils/create_reference_dumps.py  -l libhidlbase && m -j128
```
检验编译输出文件:    

```
# ls out/target/product/sargo/*.img
out/target/product/sargo/boot-debug.img     out/target/product/sargo/ramdisk.img
out/target/product/sargo/boot.img           out/target/product/sargo/ramdisk-recovery.img
out/target/product/sargo/dtb.img            out/target/product/sargo/super_empty.img
out/target/product/sargo/dtbo.img           out/target/product/sargo/system.img
out/target/product/sargo/persist.img        out/target/product/sargo/system_other.img
out/target/product/sargo/product.img        out/target/product/sargo/vbmeta.img
out/target/product/sargo/ramdisk-debug.img  out/target/product/sargo/vendor.img
```
最终编译树尺寸大小:    

```
# du -hs *
15G     android-kernel
177G    aosp10
2.4G    aosp10back
6.0G    cells
```
### 7. 烧写镜像
通过USB连接pixel 3a:    

```
$ adb devices
List of devices attached
92UAY04L95	device

$  adb reboot bootloader
```
通过以下命令烧录镜像:    

```
$ fastboot devices
92UAY04L95	fastboot
$ export ANDROID_PRODUCT_OUT=/root/Code/show/aosp10/out/target/product/sargo
$ fastboot flashall -w
--------------------------------------------
Bootloader Version...: b4s4-0.2-6355063
Baseband Version.....: g670-00042-200421-B-6414611
Serial Number........: 92UAY04L95
--------------------------------------------
Checking 'product'                                 OKAY [  0.058s]
Setting current slot to 'b'                        OKAY [  0.138s]
.....
Erasing 'userdata'                                 OKAY [  0.311s]
Erase successful, but not automatically formatting.
File system type raw not supported.
Erasing 'metadata'                                 OKAY [  0.006s]
Erase successful, but not automatically formatting.
File system type raw not supported.
Rebooting                                          OKAY [  0.001s]
Finished. Total time: 336.662s
```
将二进制驱动烧录进真机，从而可以使用无线连接:    

```
# 禁用校验机制
adb root
adb disable-verity
adb shell sync
adb reboot

# 上传模块
adb root
adb remount
adb push /media/nfs1/android-kernel/out/android-msm-pixel-4.9/dist/wlan.ko /vendor/lib/modules/

# 重启手机生效
adb reboot

```
### 8. 基本使用场景
检验编译串号:   
![/images/2022_02_03_10_18_02_498x265.jpg](/images/2022_02_03_10_18_02_498x265.jpg)

`控制中心`是用来切换双系统的快捷方式:   

![/images/2022_02_03_10_26_40_346x397.jpg](/images/2022_02_03_10_26_40_346x397.jpg)

点击`控制中心`后将出现`Start`按钮:   

![/images/2022_02_03_10_27_56_236x459.jpg](/images/2022_02_03_10_27_56_236x459.jpg)

点击`Start`按钮后，第二系统将启动:    

![/images/2022_02_03_10_28_34_234x472.jpg](/images/2022_02_03_10_28_34_234x472.jpg)

启动成功:    

![/images/2022_02_03_10_29_19_233x477.jpg](/images/2022_02_03_10_29_19_233x477.jpg)

再次点击`Start`按钮进入到第二系统:    

![/images/2022_02_03_10_30_12_230x475.jpg](/images/2022_02_03_10_30_12_230x475.jpg)

在第二系统中点击`控制中心`将提示推出该系统并回到主系统:   

![/images/2022_02_03_10_31_09_231x479.jpg](/images/2022_02_03_10_31_09_231x479.jpg)

退出后将回到第一系统的锁屏界面:    

![/images/2022_02_03_10_31_44_235x479.jpg](/images/2022_02_03_10_31_44_235x479.jpg)
### 9. 定制化
定制化1： 语言设置/输入法设置

定制化2：为双系统各自安装软件仓库:    

```
$ adb install apks/MobileAssistant_1.apk 
Performing Streamed Install
Success
```
定制化3：为双系统安装更多的apks：  

定制化4: 更改主题等
