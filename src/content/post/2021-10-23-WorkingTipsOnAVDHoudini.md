+++
title= "WorkingTipsOnAVDHoudini"
date = "2021-10-23T10:27:08+08:00"
description = "WorkingTipsOnAVDHoudini"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 添加lunch属性
添加一个`sdk_phone_x86_64-userdebug`的属性:    

```
$ vim build/envsetup.sh
....
add_lunch_combo sdk_phone_x86_64-userdebug
```
之后在lunch的时候就可以选择了。

### 更改vendor分区尺寸
更改`BOARD_VENDORIMAGE_PARTITION_SIZE`的大小:    

```
vim build/make/target/board/generic_x86_64/BoardConfig.mk
...
BOARD_VENDORIMAGE_PARTITION_SIZE := 400000000
...
```
而后重新开始编译即可:    

```
$ make installclean
$ m -j80
```
从0开始编译的命令:    

```
source build/envsetup
lunch sdk_phone_x86_64-userdebug
m -j80
```
![/images/2021_10_23_11_11_00_391x702.jpg](/images/2021_10_23_11_11_00_391x702.jpg)

### 直接更改镜像
mount到某个挂载目录:    

```
sudo mount -o loop,offset=1048576 system.img /mnt
```

### 实验步骤
初始化的r44镜像：   

![/images/2021_10_23_11_12_25_499x281.jpg](/images/2021_10_23_11_12_25_499x281.jpg)

按照houdini的容器方案替代:

### 换方法
使用另外一种android发行版:    

```
lineage-16.0
lineage-17.1
lineage-18.1
```
