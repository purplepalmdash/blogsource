+++
title= "WorkingTipsOnWinInDocker"
date = "2021-04-20T10:51:25+08:00"
description = "WorkingTipsOnWinInDocker"
keywords = ["Technology"]
categories = ["Technology"]
+++

### 制作Windows镜像
CentOS7上以以下方式启动虚拟机:   

```
/usr/libexec/qemu-kvm -enable-kvm \
        -machine q35 -smp sockets=1,cores=1,threads=2 -m 2048 \
        -usb -device usb-kbd -device usb-tablet -rtc base=localtime \
        -net nic,model=virtio -net user,hostfwd=tcp::4444-:4444 \
        -drive file=hdd.img,media=disk,if=virtio \
        -drive file=/home/docker/win/cn_windows_10_consumer_editions_version_2004_x64_dvd.iso,media=cdrom \
        -drive file=/home/docker/win/virtio-win-0.1.141.iso,media=cdrom
```
用qemu提示的vnc端口访问该运行中的实例:    

![/images/2021_04_20_10_53_00_636x464.jpg](/images/2021_04_20_10_53_00_636x464.jpg)

选择`自定义安装`:   

![/images/2021_04_20_10_53_46_593x403.jpg](/images/2021_04_20_10_53_46_593x403.jpg)

需加载驱动程序:   

![/images/2021_04_20_10_55_28_488x496.jpg](/images/2021_04_20_10_55_28_488x496.jpg)

选择好后的驱动:   

![/images/2021_04_20_10_55_56_414x92.jpg](/images/2021_04_20_10_55_56_414x92.jpg)

忽略警告，继续:    

![/images/2021_04_20_10_56_46_553x286.jpg](/images/2021_04_20_10_56_46_553x286.jpg)

继续安装直到安装完毕。   

![/images/2021_04_20_14_23_46_568x429.jpg](/images/2021_04_20_14_23_46_568x429.jpg)

密码:   

![/images/2021_04_20_14_25_14_533x423.jpg](/images/2021_04_20_14_25_14_533x423.jpg)

更新驱动程序:    

![/images/2021_04_20_14_40_16_616x407.jpg](/images/2021_04_20_14_40_16_616x407.jpg)

选中E:\后更新:   

![/images/2021_04_20_14_41_40_548x273.jpg](/images/2021_04_20_14_41_40_548x273.jpg)

此时关闭vm, 并创建一个overlay的image并使用该image启动一次vm:    

```
$ qemu-img create -b hdd.img -f qcow2 snapshot.img
$ /usr/libexec/qemu-kvm -enable-kvm \
        -machine q35 -smp sockets=1,cores=1,threads=2 -m 2048 \
        -usb -device usb-kbd -device usb-tablet -rtc base=localtime \
        -net nic,model=virtio -net user,hostfwd=tcp::4444-:4444 \
        -drive file=snapshot.img,media=disk,if=virtio \
        -monitor stdio
```
在qemu终端内， 保存当前的状态后关机:    

```
(qemu) savevm windows
Then type quit to stop VM:

(qemu) quit
```
因为有save后的状态，因而如果我们能保证容器内的qemu与容器外的qemu是同一版本的话，则可以快速恢复。    

### 编译容器镜像

```
$ mv hdd.img snapshot.img image
$ cd image
$ docker build -t windows/win10qemu:20210420 .
```
在Centos7系列的操作系统上，因为宿主机的qemu版本与容器中的qemu版本差异，导致无法启动，需做以下修改:    

```
# vim entrypoint.sh
....

  qemu-system-x86_64 -enable-kvm \
    -machine q35 -smp sockets=1,cores=1,threads=2 -m 2048 \
    -usb -device usb-kbd -device usb-tablet -rtc base=localtime \
    -net nic,model=virtio -net user,hostfwd=tcp::4444-:4444 \
    -drive file=snapshot.img,media=disk,if=virtio &
...
# vim Dockerfile
FROM windows/win10qemu:20210420
COPY entrypoint.sh /
# docker build -t win/win10new:latest .
```
运行容器:    

```
# docker run -it --rm --privileged -p 4444:4444 -p 5915:5900  win/win10new:latest
```
打开vnc软件开始访问5915端口可以看到Windows桌面:    

![/images/2021_04_20_16_56_36_779x615.jpg](/images/2021_04_20_16_56_36_779x615.jpg)

### K8s中运行
由容器镜像创建出pod负载，service暴露即可。
