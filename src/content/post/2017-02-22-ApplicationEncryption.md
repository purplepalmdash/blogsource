+++
date = "2017-02-22T09:35:12+08:00"
title = "应用框架加密"
categories = ["Linux"]
keywords = ["Encryption"]
description = "Encryption of the application frameworks"

+++
### 目的
在诸如OpenSuse
11.1这一类的古老系统上，对组建系统的加密可能会比较繁琐，首先全盘加密的实现并不容易(几乎不可能，因为内核太古老)。

### 测试用框架
为了简单起见，我们将使用一个非常简单的基于C语言的网页服务器，来将我们的静态网站导出，静态网站也没什么内容，大体截图如下:    

![/images/2017_02_22_09_42_53_811x582.jpg](/images/2017_02_22_09_42_53_811x582.jpg)    

框架目录结构如下:    

```
.
├── images
│   └── 2017_02_22_09_41_24_1599x874.jpg
├── index.html
└── index.md

1 directory, 3 files
```
### 网页服务器
为了测试的方便，我们用一个基于C实现的静态网页服务器:    

```
$ wget https://gist.githubusercontent.com/sumpygump/9908417/raw/5fa991fda103d0b7a0c38512394a83ccada9ad6c/nweb23.c
$ gcc -O -DLINUX nweb32.c -o nweb
$ ./nweb 8848 ./
```
加入到启动项中:    

```
$ sudo vim /etc/rc.d/boot.local
/home/dash/Code/nweb 8848 /home/dash/testwebsite/ &
```
现在打开浏览器访问该机器的8848端口，可以看到我们的框架已经正确运行:    

![/images/2017_02_22_10_02_40_784x686.jpg](/images/2017_02_22_10_02_40_784x686.jpg)    

### 加密问题
现在的问题在于：一旦用户登入到系统，对我们框架的结构即可一目了然。   

因而首先要解决的是：禁止用户登入进LINUX系统，这个步骤很简单，设置用户名/密码更加复杂就可以，理论上可以防止用户暴力破解并登入系统。    

但是一旦用户拔下硬盘，将此机器上的硬盘插入到另一台机器上，则框架的所有结构同样一览无余，接下来我们来解决框架的加密问题.    

### 虚拟加密盘
创建一个大小为1G的虚拟加密盘raw文件:    

```
# dd if=/dev/zero of=/root/luks.vol bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 9.32633 s, 115 MB/s
```
当11.1系统运行加密的时候，会得到以下错误:    

```
# cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 10000 luksFormat /root/luks.vol 

WARNING!
========
This will overwrite data on /root/luks.vol irrevocably.

Are you sure? (Type uppercase yes): YES
Enter LUKS passphrase: 
Verify passphrase: 
Command failed: Unable to obtain sector size for /root/luks.vol
```
而在12.3系统上，则加密成功,猜想应该是cryptsetup版本的问题:    

```
 # cryptsetup --cipher aes-xts-plain64 --key-size 512 --hash sha512 --iter-time 10000 luksFormat /root/luks.vol

WARNING!
========
This will overwrite data on /root/luks.vol irrevocably.

Are you sure? (Type uppercase yes): YES
Enter LUKS passphrase: 
Verify passphrase: 
```
### 自动挂载加密盘
自动挂在加密盘可以通过keyfile来进行，创建一个用于解锁加密raw文件的keyfile的步骤如下:    

```
# dd if=/dev/urandom of=/root/file_key bs=512 count=8
8+0 records in
8+0 records out
4096 bytes (4.1 kB) copied, 0.000364836 s, 11.2 MB/s
# cryptsetup -v luksAddKey /root/luks.vol /root/file_key
Enter any passphrase: 
Key slot 0 unlocked.
Command successful.
```
创建一个用于挂载加密raw文件的位置:    

```
# mkdir -p /media/vol
```
用keyfile解锁并挂载到`/media/vol`:    

```
# cryptsetup -v luksOpen /root/luks.vol vol_crypt --key-file=/root/file_key
# mkfs.ext4 /dev/mapper/vol_crypt
# mount /dev/mapper/vol_crypt /media/vol
```
查看是否被挂载成功:   

```
# df -h | grep crypt
/dev/mapper/vol_crypt 1006M   18M  938M   2% /media/vol
```

将框架拷贝入加密目录:    

```
# cp -r testwebsite/ /media/vol/
# cp /root/nweb /media/vol/
```

现在在OpenSuse的自启动文件中添加以下条目，用于自动解锁及开机时自动运行框架:    

```
# vim /etc/rc.d/boot.local
cryptsetup -v luksOpen /root/luks.vol vol_crypt --key-file=/root/file_key
mount /dev/mapper/vol_crypt /media/vol
/media/vol/nweb 8848 /media/vol/testwebsite/ &
```
重新启动机器，验证结果:    

```
# ps -ef | grep nweb
root      1980     1  0 21:52 ?        00:00:00 /media/vol/nweb 8848 /media/vol/testwebsite/
root      1983  1876  0 21:52 pts/0    00:00:00 grep --color=auto nweb
```
打开浏览器，可以看到该框架已经被正确运行, 而这一次框架的可执行文件位于加密磁盘里。   

### 试图解锁框架
前面说过，如果用户能登入系统，则框架对用户来说完全可见，不存在加密的问题。所以第一步应该是阻止用户登录入系统.    

如果用户得到虚拟机磁盘，挂载到另一台机器上：    

![/images/2017_02_22_11_10_13_606x347.jpg](/images/2017_02_22_11_10_13_606x347.jpg)    

启动后检查磁盘的挂载情况, 可以看到40G 大小的OpenSuse12.3系统盘已经被挂载上:     

```
$ sudo fdisk -l /dev/vdb
Disk /dev/vdb: 40 GiB, 42949672960 bytes, 83886080 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x0000bea3

Device     Boot    Start      End  Sectors  Size Id Type
/dev/vdb1           2048  4208639  4206592    2G 82 Linux swap / Solaris
/dev/vdb2  *     4208640 36403199 32194560 15.4G 83 Linux
/dev/vdb3       36403200 83886079 47482880 22.7G 83 Linux
```
`/dev/vdb2`为OpenSuse 12.3的系统盘，我们挂载一下，并检查`/media/vol`里的内容:    

```
# mount /dev/vdb2 /mnt1
# ls /mnt1
bin  boot  dev  encryption  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  selinux  srv  sys  tmp  usr  var
# ls /mnt1/media/vol/
```
可以看到此挂载目录下框架内容完全不可见，而此时如果硬行查看`/root/luks.vol`文件，只能看到乱码.    

解密加密后的raw文件的关键在于拿到keyfile,
而我们的keyfile位于`/root/keyfile`，理论上还是存在被解密的可能。为了保密起见，可以考虑将此keyfile置于可移动介质中，或者云端。
