+++
date = "2017-01-22T18:31:43+08:00"
keywords = ["PasswordlessEncryption"]
description = "Passwordless Encryption For debian"
title = "使用USB磁盘无密码登录加密分区的LINUX"
categories = ["Linux"]

+++
### 背景
最近一直在看关于LINUX磁盘加密的东西，也尝试了文件加密、扇区加密、全盘加密等方面。
然而每次都需要输入密码这种事情对一个追求速度的人来说似乎有点浪费时间，而且每次重新
都需要输入密码也无法适用于自动化运维的场合。因而我根据网上提到的方法制作了一个
从USB磁盘读取密钥以解密加密全盘的方法。下面是对应的实现过程。    

### Debian系统安装
从镜像网站下载Debian
8(jessie)的DVD安装源，在Virtualbox中创建一台虚拟机，插入
ISO后，光盘启动系统进入到以下界面:    

![/images/2017_01_22_18_52_55_444x325.jpg](/images/2017_01_22_18_52_55_444x325.jpg)

选择`Install`后，继续安装，因为篇幅的关系，这里省略掉关于用户名/密码/主机名/语言/区域设置等配置，
直接进入到磁盘分区的步骤:    

选择手动对磁盘进行分区:    

![/images/2017_01_22_18_54_29_556x251.jpg](/images/2017_01_22_18_54_29_556x251.jpg)
选择我们添加的磁盘:    

![/images/2017_01_22_18_55_13_547x201.jpg](/images/2017_01_22_18_55_13_547x201.jpg)

选择接受上面的提示，创建一个新的空白分区表:    

![/images/2017_01_22_18_56_33_538x154.jpg](/images/2017_01_22_18_56_33_538x154.jpg)

可以看到当前没有任何磁盘分区，选择到红色光标所在的行，按回车:    

![/images/2017_01_22_18_57_25_543x255.jpg](/images/2017_01_22_18_57_25_543x255.jpg)

创建一个新的分区:    

![/images/2017_01_22_18_57_55_388x178.jpg](/images/2017_01_22_18_57_55_388x178.jpg)

`/boot`分区大小我们设置为400MB：    

![/images/2017_01_22_18_58_16_541x168.jpg](/images/2017_01_22_18_58_16_541x168.jpg)

选择为主分区:    

![/images/2017_01_22_18_58_33_301x156.jpg](/images/2017_01_22_18_58_33_301x156.jpg)

Location选择，从Beginning开始:    

![/images/2017_01_22_18_59_04_397x114.jpg](/images/2017_01_22_18_59_04_397x114.jpg)

配置为我们以下截图里所示的内容:    

![/images/2017_01_22_18_59_37_452x219.jpg](/images/2017_01_22_18_59_37_452x219.jpg)

接着创建一个大小为1GB的交换分区:    

![/images/2017_01_22_19_00_12_456x214.jpg](/images/2017_01_22_19_00_12_456x214.jpg)

![/images/2017_01_22_19_00_22_530x167.jpg](/images/2017_01_22_19_00_22_530x167.jpg)

![/images/2017_01_22_19_00_51_545x199.jpg](/images/2017_01_22_19_00_51_545x199.jpg)

接着创建主分区，步骤不再重复，充分利用剩下的磁盘空间:    

![/images/2017_01_22_19_01_31_411x192.jpg](/images/2017_01_22_19_01_31_411x192.jpg)

在上面的高亮区内按回车，改变磁盘类型为`physical volume for encryption`:    

![/images/2017_01_22_19_01_59_326x335.jpg](/images/2017_01_22_19_01_59_326x335.jpg)

注意我们配置的参数，这里配置了默认的加密方式为device-mapper(dm-crypt)，
加密算法为256键字节大小的AES加密:    

![/images/2017_01_22_19_03_07_517x251.jpg](/images/2017_01_22_19_03_07_517x251.jpg)

成功创建后的磁盘分区情况如下图:    

![/images/2017_01_22_19_03_50_428x209.jpg](/images/2017_01_22_19_03_50_428x209.jpg)

接下来对加密卷作高阶配置，选择如下图中的条目:    

![/images/2017_01_22_19_06_47_321x159.jpg](/images/2017_01_22_19_06_47_321x159.jpg)

出现警告信息，告知我们当前的分区方案将被写入到磁盘，我们是否开始配置加密卷？    

![/images/2017_01_22_19_08_03_435x207.jpg](/images/2017_01_22_19_08_03_435x207.jpg)

选择yes后，选择`Create encrypted volumes`:    

![/images/2017_01_22_19_08_41_456x182.jpg](/images/2017_01_22_19_08_41_456x182.jpg)

我们选择sda2和sda3，对交换分区和主分区进行加密。如果只选择主分区加密，安装程序会
报出错误，所以一开始我们就选中这两块分区，进入到下一步:    

![/images/2017_01_22_19_09_34_447x245.jpg](/images/2017_01_22_19_09_34_447x245.jpg)

分区完毕，选择`Done Setting up the partition`， 结束手动分区步骤:    

![/images/2017_01_22_19_10_13_399x212.jpg](/images/2017_01_22_19_10_13_399x212.jpg)

确定写入磁盘:    

![/images/2017_01_22_19_10_29_542x135.jpg](/images/2017_01_22_19_10_29_542x135.jpg)

写入过程:    

![/images/2017_01_22_19_10_50_553x320.jpg](/images/2017_01_22_19_10_50_553x320.jpg)

此时需要定义密码:    

![/images/2017_01_22_19_11_12_534x209.jpg](/images/2017_01_22_19_11_12_534x209.jpg)

成功配置完加密卷的磁盘分区表如下所示:    

![/images/2017_01_22_19_11_55_510x295.jpg](/images/2017_01_22_19_11_55_510x295.jpg)

选择4G大小的主分区，改变其Mount point为/:    

![/images/2017_01_22_19_12_29_545x266.jpg](/images/2017_01_22_19_12_29_545x266.jpg)

![/images/2017_01_22_19_12_41_551x245.jpg](/images/2017_01_22_19_12_41_551x245.jpg)

同样改变swap分区:    

![/images/2017_01_22_19_12_55_499x159.jpg](/images/2017_01_22_19_12_55_499x159.jpg)

确定写入磁盘:    

![/images/2017_01_22_19_13_06_540x222.jpg](/images/2017_01_22_19_13_06_540x222.jpg)

接下来继续安装，并在安装的最后一步将BootLoad写入MBR。   

安装完毕后，重新启动机器后，需要输入密码才能进入系统:    

![/images/2017_01_22_19_14_22_453x97.jpg](/images/2017_01_22_19_14_22_453x97.jpg)

![/images/2017_01_22_19_14_32_463x151.jpg](/images/2017_01_22_19_14_32_463x151.jpg)

### 配置passwordless
接下来我们创建无需输入密码的根文件系统，使用一个USB优盘来记录密码信息，位于USB优盘
上的secret key将用于解密加密磁盘。
将USB盘插入到虚拟机，可以使用dmesg来获取其卷标，这里可以看到我们的卷表为`/dev/sdb`:    

![/images/2017_01_22_19_16_35_540x296.jpg](/images/2017_01_22_19_16_35_540x296.jpg)

使用dd命令，从优盘读取8192字节大小的随机字节 ，作为我们要使用的secret key:    

```
# dd if=/dev/sdb of=/root/secret.key bs=512 skip=4 count=16
```
上面生成的secret.key文件将被加入到加密卷中，通过`cryptsetup`命令加入。默认的密码被保存
在slot 0, 而slot 1将被用作第二个secret key。    

首先使用blkid命令来查看磁盘卷的详细信息:    

```
# blkid 
/dev/mapper/sda3_crypt: UUID="6d978850-19f3-4a89-87cb-01825d7601c4" TYPE="ext4"
/dev/sda1: UUID="3288f26a-7669-44fe-b130-87d39ab57166" TYPE="ext4" PARTUUID="f7870f8a-01"
/dev/sda2: PARTUUID="f7870f8a-02"
/dev/sda3: UUID="8ca94b2f-89d3-4114-9e33-290dc57ed723" TYPE="crypto_LUKS" PARTUUID="f7870f8a-03"
/dev/sdb1: LABEL="XENSERVER-6" UUID="5CD6-02A1" TYPE="vfat" PARTUUID="0003fa4f-01"
```
被用于解密卷的secret key将仅仅被加入到`/dev/sda3`中，当然，它也可以被加入到`/dev/sda2`中。    

```
# cryptsetup luksAddKey /dev/sda3 /root/secret.key --key-slot 1
Enter any passphrase!
```
我们将创建一个简单的udev规则，用于创建USB设备，我们将创建一个`/etc/udev/rules.d/99-custom-usb.rules`
文件，并将它的链接文件指向`/dev/usbdevice`:    

```
# vi /etc/udev/rules.d/99-custom-usb.rules 
SUBSYSTEMS=="usb",DRIVERS=="usb",SYMLINK+="usbdevice%n"
```
重新加载udev规则:    

```
# udevadm control --reload-rules
```
现在插入磁盘后，以验证自定义规则:    

```
# ls -l /dev/usbdevice
lrwxrwxrwx 1 root root 3 Jan 22 04:27 /dev/usbdevice -> sdb
```
创建一个shell脚本，用于在系统启动时，从USB设备读取出secret key，并将其
提供给cryptsetup。 这个脚本位于`/usr/local/sbin/openluksdevices.sh`:    

```SHELL
#!/bin/sh
############taken from following link#########
###http://www.oxygenimpaired.com/debian-lenny-luks-encrypted-root-hidden-usb-keyfile

TRUE=0
FALSE=1

# flag tracking key-file availability
OPENED=$FALSE

if [ -b /dev/usbdevice ]; then
# if device exists then output the keyfile from the usb key 
dd if=/dev/usbdevice bs=512 skip=4 count=16 | cat
OPENED=$TRUE
fi

if [ $OPENED -ne $TRUE ]; then
echo "FAILED to get USB key file ..." >&2
/lib/cryptsetup/askpass "Try LUKS password: "
else
echo "Success loading key file for Root . Moving on." >&2
fi

sleep 2
```
添加可执行权限:    

```
# chmod a+x /usr/local/sbin/openluksdevices.sh
```
与fstab文件类似， crypttab文件包含有在Linux平台加密卷的相关配置信息. 这里
我们添加针对`sda3_crypt`加密卷的配置信息:    

```
# vi /etc/crypttab 
sda2_crypt UUID=4270c1eb-5a06-4a1f-8a19-d5af9db4f779 none luks,swap
sda3_crypt UUID=8ca94b2f-89d3-4114-9e33-290dc57ed723 none luks,keyscript=/usr/local/sbin/openluksdevices.sh
```
同样，我们需要重新生成initramfs，因而在initramfs的制作工具的定义文件里
我们也需要添加针对`sda3_crypt`卷的解密操作:    

```
# vi /etc/initramfs-tools/conf.d/cryptroot 
CRYPTROOT=target=sda3_crypt,source=/dev/disk/by-uuid/8ca94b2f-89d3-4114-9e33-290dc57ed723
```
并在`/etc/initramfs-tools`中添加`usb_storage`选项:    

```
# vi /etc/initramfs-tools/modules 
usb_storage
```
添加`udevusbkey.sh`文件，以在临时文件系统`initrd`中添加自定义的udev规则:    

`/etc/initramfs-tools/hooks/udevusbkey.sh `文件:    

```
#!/bin/sh
# udev-usbkey script
###taken from
###http://www.oxygenimpaired.com/ubuntu-with-grub2-luks-encrypted-lvm-root-hidden-usb-keyfile
PREREQ="udev"
prereqs()
{
echo "$PREREQ"
}

case $1 in
prereqs)
prereqs
exit 0
;;
esac

. /usr/share/initramfs-tools/hook-functions

# Copy across relevant rules

cp /etc/udev/rules.d/99-custom-usb.rules ${DESTDIR}/lib/udev/rules.d/

exit 0
```
并给予其可执行权限:    

```
#  chmod a+x /etc/initramfs-tools/hooks/udevusbkey.sh
```
GRUB2配置文件也需要进行相应的修改，以添加`rootdelay`和`cryptopts`选项:    

```
$ vi /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="rootdelay=20 cryptopts=target=sda3_crypt,source=/dev/disk/by-uuid/8ca94b2f-89d3-4114-9e33-290dc57ed723,keyscript=/lib/cryptsetup/scripts/openluksdevices.sh"
GRUB_CMDLINE_LINUX=""
```
运行`update-grub`以更新grub配置文件，可以通过检查`/boot/grub/grub.cfg`来看到我们更改
后的配置已经被加入到最终配置文件。

运行`update-initramfs -u`来更新系统启动时所需用到的临时文件系统.    

运行检查程序，查看openlunksdevices是否被正确打包入initramfs中:    

```
# cd /tmp
# zcat /boot/initrd.img-3.16.0-4-amd64 | cpio -iv
# ls -l lib/cryptsetup/scripts/openluksdevices.sh 
-rwxr-xr-x 1 root staff 565 Jan 22 06:52 lib/cryptsetup/scripts/openluksdevices.sh
```
最后的启动过程可以参考:    

![/images/2017_01_22_19_54_12_548x301.jpg](/images/2017_01_22_19_54_12_548x301.jpg)

### 总结
这里使用一个USB存储设备打开了LINUX系统里的一个加密盘，一个自动化脚本
被用来在启动时为机密卷提供一个用于解密的secret key.    

### Bug
按照原教程里的配置，导致swap分区无法被激活，我觉得可能需要添加关于swap分区的选项。    

更新，在分区指定swap分区加密的时候，选定密码类型为random password.     
