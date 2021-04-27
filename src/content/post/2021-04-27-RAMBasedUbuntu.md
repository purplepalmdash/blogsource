+++
title= "完全用RAM运行Ubuntu"
date = "2021-04-27T09:54:18+08:00"
description = "RAMBasedUbuntu"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. 目的
将Ubuntu18.04.1操作系统(arm64)完全运行在内存中。   

### 2. 准备材料
Ubuntu 18.04.1 arm64安装iso.    
arm64服务器/libvirtd/virt-manager.(在没有实体服务器的情况下，可以用虚拟机来模拟测试).    

### 3. 步骤
最小化安装Ubuntu 18.04.1 操作系统, 根分区最好包含所有分区(all in one)。   
安装完毕操作系统后，定制自己需要的软件包及准备环境后，删除所有的临时文件，尽量瘦身系统。这是因为内存定制化后，所有的文件在启动时将被加载到内存！全新安装的ubuntu大约占据约1.5GB的磁盘空间。        
以下为定制为RAM启动的流程：   

步骤一：    
更改`/etc/fstab`文件内容，首先备份该文件:    

```
# cp /etc/fstab /etc/fstab.bak
```
编辑`/etc/fstab`文件内容，找到标识根分区(/)的行，更改为以下内容(下为示例):    

```
#/dev/mapper/ubuntu--vg-root /               ext4    errors=remount-ro 0       1
none / tmpfs defaults 0 0
```

步骤二:    
更改initramfs中的`local`脚本内容, initramfs 包含的工具和脚本，在正式的根文件系统的初始化脚本 init 启动之前，就被挂载并完成相应的初始化工作。我们需要提前将磁盘根分区中的内容拷贝入`tmpfs`中，以便在`/etc/fstab`开始执行的时候找寻到正确的分区.    

首先备份`/usr/share/initramfs-tools/scripts/local`文件:    

```
# cp /usr/share/initramfs-tools/scripts/local /usr/share/initramfs-tools/scripts/local.bak   
```

编辑`local`文件，更改其`Mount root`部分的处理逻辑(约204行左右内容):     

```
	# FIXME This has no error checking
	# Mount root
	#mount ${roflag} ${FSTYPE:+-t ${FSTYPE} }${ROOTFLAGS} ${ROOT} ${rootmnt}
	# Start of ramboottmp
        mkdir /ramboottmp
        mount ${roflag} -t ${FSTYPE} ${ROOTFLAGS} ${ROOT} /ramboottmp
        mount -t tmpfs -o size=100% none ${rootmnt}
        cd ${rootmnt}
        cp -rfa /ramboottmp/* ${rootmnt}
        umount /ramboottmp
        ### End of ramboottmp
```
保存该文件后，重新编译initramfs:    

```
# mkinitramfs -o /boot/initrd.img-ramboot
```
编译成功后，将`local`文件替换会原来的版本:    

```
# cp -f /usr/share/initramfs-tools/scripts/local.bak /usr/share/initramfs-tools/scripts/local
```

步骤三:   
更改grub，以使用刚才编译出的`initrd.img-ramboot`来启动操作系统:    

更改第一启动项中的`/initrd`行，替换为:    

```
# chmod +w /boot/grub/grub.cfg
# vim /boot/grub/grub.cfg
.....
.....
        linux	/boot/vmlinuz-4.15.0-29-generic root=/dev/mapper/ubuntu--vg-root ro  
	initrd	/boot/initrd.img-ramboot
......
......
# chmod -w /boot/grub/grub.cfg
```

步骤四：    
重启，重启时选择第一启动项，此时根分区会整体被加载到`tmpfs`中。    

### 4. 性能对比测试
测试环境定义:
* aarch64 4核
* 64 GB 内存
* 100 GB 磁盘分区
* Ubuntu 18.04.1 LTS
* 内核版本： 4.15.0-29-generic
* fio版本: fio-3.1   

所有测试样例均在`ramdisk`主机及传统主机上运行并对比.    

#### 4.1 fio 4k随机读写
测试命令如下:    

```
# fio --name TEST --eta-newline=5s --filename=fio-tempfile.dat --rw=randrw --size=500m --io_size=10g --blocksize=4k --ioengine=libaio --fsync=1 --iodepth=1 --numjobs=1 --runtime=60 --group_reporting
```

|指标 | 内存型主机 | 传统主机 |
| ----------- | ----------- | --------- |
| READ bw | bw=513MiB/s (538MB/s) | bw=85.0KiB/s (87.0kB/s) |
| READ io | io=5133MiB (5382MB) | io=5104KiB (5226kB) |
| READ iops | IOPS=131k | IOPS=21 |
| WRITE bw | bw=510MiB/s (535MB/s) | bw=88.1KiB/s (90.2kB/s) |
| WRITE io | io=5107MiB (5355MB) | io=5288KiB (5415kB) |
| WRITE iops | IOPS=131k | IOPS=22 |

测试显示：4K随机读写的带宽对比，内存型主机是传统主机的约6000倍，读IOPS/写IOPS，内存型主机是传统主机的约6000倍。    
#### 4.2 fio 4k顺序读写
测试命令如下:    

```
# fio --name TEST --eta-newline=5s --filename=fio-tempfile.dat --rw=rw --size=500m --io_size=10g --blocksize=4k --ioengine=libaio --fsync=1 --iodepth=1 --numjobs=1 --runtime=60 --group_reporting
```

|指标 | 内存型主机 | 传统主机 |
| ----------- | ----------- | --------- |
| READ bw | bw=640MiB/s (671MB/s) | bw=73.2KiB/s (75.0kB/s) |
| READ io | io=5133MiB (5382MB) | io=4396KiB (4502kB) |
| READ iops | IOPS=164k | IOPS=18 |
| WRITE bw | bw=637MiB/s (668MB/s) | bw=76.8KiB/s (78.6kB/s) |
| WRITE io | io=5107MiB (5355MB) | io=4608KiB (4719kB) |
| WRITE iops | IOPS=163k | IOPS=19 |

测试显示：4K顺序读写的带宽对比，内存型主机是传统主机的约9000倍，读IOPS/写IOPS，内存型主机是传统主机的约9000倍。    
