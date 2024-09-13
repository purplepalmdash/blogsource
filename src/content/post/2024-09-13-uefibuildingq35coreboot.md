+++
title= "uefibuildingq35coreboot"
date = "2024-09-13T10:24:38+08:00"
description = "uefibuildingq35coreboot"
keywords = ["Technology"]
categories = ["Technology"]
+++
安装以下依赖包:    

```
 sudo apt install -y build-essential git vim m4 bison flex zlib1g-dev libncurses5-dev intltool libtool gperf libcap-dev libblkid-dev libmount-dev xsltproc docbook-xsl autopoint libgpgme11-dev libdevmapper-dev libdw-dev libdw1 libssl-dev libevent-dev
```
创建编译目录:    

```
mkdir Code
mkdir  -p Code/coreboot
cd Code/coreboot
 git clone http://review.coreboot.org/p/coreboot
 cd coreboot/
 git checkout tags/4.6 -b local46
 wget https://fossies.org/linux/misc/old/libelf-0.8.13.tar.gz
mkdir -p util/crossgcc
 mv libelf-0.8.13.tar.gz util/crossgcc/tarballs/
 vim util/crossgcc/buildgcc 
IASL_ARCHIVE="https://downloadmirror.intel.com/774735/acpica-unix2-${IASL_VERSION}.tar.gz"
 make crossgcc CPUS=`nproc`
 make iasl CPUS=`nproc`
 make menuconfig
```
编译systemd, 注意这里使用了特定版本的systemd:     

```
cd ~/Code
mkdir systemd
cd systemd/
wget https://github.com/systemd/systemd/archive/refs/tags/v229.tar.gz
tar xzvf v229.tar.gz 
mv systemd-229/ systemd
cd systemd/
./autogen.sh
mkdir build
cd build
../configure --prefix=/usr --enable-blkid --disable-seccomp --disable-libcurl --disable-pam --disable-kmod
make -j12
cd ../../../
```
编译kexec:    

```
mkdir kexec
cd kexec
git clone git://git.kernel.org/pub/scm/utils/kernel/kexec/kexec-tools.git
cd kexec-tools
./bootstrap
./configure --prefix=/usr
vim /home/dash/Code/kexec/kexec-tools/kexec/arch/i386/x86-linux-setup.c
      /#include <sys/random.h>
      #include <linux/random.h>
      #include <unistd.h>
      #include <sys/syscall.h>
......
	//if (getrandom(sd->rng_seed, sizeof(sd->rng_seed), GRND_NONBLOCK) !=
	if (syscall(SYS_getrandom,sd->rng_seed, sizeof(sd->rng_seed), GRND_NONBLOCK) !=
make -j`nproc`
cd ../..
```
编译twin:    

```
mkdir petitboot
cd petitboot/
git clone git://git.kernel.org/pub/scm/linux/kernel/git/geoff/libtwin.git
cd libtwin/
cp README.md README
./autogen.sh  && make -j8 && sudo make install
cd ../../
```
编译petitboot:    

```
cd petitboot/
wget https://git.raptorengineering.com/git/petitboot/snapshot/petitboot-1.4.3.tar.gz
tar xzvf petitboot-1.4.3.tar.gz 
mv petitboot-1.4.3/ petitboot
cd petitboot/
./bootstrap 
CPPFLAGS="-I../../systemd/systemd/src/libudev/" LDFLAGS="-L../../systemd/systemd/build/.libs/" ./configure --prefix=/usr --enable-static --disable-shared --enable-busybox --with-ncurses --without-twin-x11 --without-twin-fbdev --with-signed-boot
make -j12
cd ../../
```
编译busybox:    

```
mkdir busybox
cd busybox
git clone git://git.busybox.net/busybox
cd busybox
make defconfig
make menuconfig
LDFLAGS=--static make -j`nproc`
cd ../.. 
```
make menuconfig时，去掉下面这个选项:     

![/images/20240913_104714_x.jpg](/images/20240913_104714_x.jpg)

现在需要编译一个最小化的initramfs， 准备基本的目录架构:     

```
mkdir initramfs
mkdir -p initramfs/{bin,sbin,etc,lib,proc,sys,newroot,usr,usr/bin,usr/sbin,var,var/log,run,run/udev,tmp}
mkdir initramfs/var/log/petitboot
touch initramfs/etc/mdev.conf
cp -Rp /lib/terminfo initramfs/lib/
cp -Rp busybox/busybox/busybox initramfs/bin/
ln -s busybox initramfs/bin/sh
```
从本机上拷贝核心库:     

```
mkdir -p initramfs/lib/x86_64-linux-gnu
cp -L /lib/x86_64-linux-gnu/libc.so.* initramfs/lib/x86_64-linux-gnu/
cp -L /lib/x86_64-linux-gnu/libm.so.* initramfs/lib/x86_64-linux-gnu/
cp -L /lib/x86_64-linux-gnu/libdl.so.* initramfs/lib/x86_64-linux-gnu/
cp -L /lib/x86_64-linux-gnu/librt.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libacl.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libcap.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libattr.so.* initramfs/lib/x86_64-linux-gnu/
cp -L /lib/x86_64-linux-gnu/libpthread.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libncurses.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libtinfo.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libpcre.so.* initramfs/lib/x86_64-linux-gnu/
cp -L /lib/x86_64-linux-gnu/libresolv.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libselinux.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libreadline.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libgcc_s.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libblkid.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libkmod.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libuuid.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libusb-0.1.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libdevmapper.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libz.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/liblzma.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libbz2.so.* initramfs/lib/x86_64-linux-gnu/
cp -R /lib/x86_64-linux-gnu/libgpg-error.so.* initramfs/lib/x86_64-linux-gnu/
cp -L /lib/x86_64-linux-gnu/libnss_files.so.* initramfs/lib/x86_64-linux-gnu/

mkdir -p initramfs/lib64/
cp -L /lib64/ld-linux-x86-64.so.* initramfs/lib64/

mkdir -p initramfs/usr/lib/x86_64-linux-gnu/
cp -R /usr/lib/x86_64-linux-gnu/libform.so.* initramfs/usr/lib/x86_64-linux-gnu/
cp -R /usr/lib/x86_64-linux-gnu/libmenu.so.* initramfs/usr/lib/x86_64-linux-gnu/
cp -L /usr/lib/x86_64-linux-gnu/libelf.so.* initramfs/usr/lib/x86_64-linux-gnu/
cp -L /usr/lib/x86_64-linux-gnu/libdw.so.* initramfs/usr/lib/x86_64-linux-gnu/
cp -R /usr/lib/x86_64-linux-gnu/libgpgme.so.* initramfs/usr/lib/x86_64-linux-gnu/
cp -R /usr/lib/x86_64-linux-gnu/libassuan.so.* initramfs/usr/lib/x86_64-linux-gnu/
```
复制辅助类的二进制文件到新的initramfs中:     

```
cp -Rp /usr/bin/gpg initramfs/usr/bin/

cp systemd/systemd/build/.libs/libudev.so.* initramfs/lib/x86_64-linux-gnu/
cp -Rp systemd/systemd/build/systemd-udevd initramfs/sbin/
cp -Rp systemd/systemd/build/udevadm initramfs/sbin/

mkdir -p initramfs/usr/lib/udev
cp -Rp systemd/systemd/build/*_id initramfs/usr/lib/udev

cp -Rp kexec/kexec-tools/build/sbin/kexec initramfs/sbin/
```
安装petitboot到新的initramfs中:     

```
cd petitboot/petitboot
make DESTDIR=`realpath ../../initramfs/` install
cd ../..
```
拷贝udev规则到新的initramfs中:      

```
mkdir -p initramfs/usr/lib/udev/rules.d
cp -Rp systemd/systemd/rules/* initramfs/usr/lib/udev/rules.d/
cp -Rp systemd/systemd/build/rules/* initramfs/usr/lib/udev/rules.d/
rm -f initramfs/usr/lib/udev/rules.d/*-drivers.rules
```
设置udhcp辅助脚本:     

```
mkdir -p initramfs/usr/share/udhcpc/
cp -Rp busybox/busybox/examples/udhcp/simple.script initramfs/usr/share/udhcpc/simple.script
chmod 755 initramfs/usr/share/udhcpc/simple.script

sed -i '/should be called from udhcpc/d' initramfs/usr/share/udhcpc/simple.script

cat << EOF > initramfs/usr/share/udhcpc/default.script
#!/bin/sh

/usr/share/udhcpc/simple.script "\$@"
/usr/sbin/pb-udhcpc "\$@"
EOF
chmod 755 initramfs/usr/share/udhcpc/default.script
```
设置nsswitch:    

```
touch initramfs/etc/nsswitch.conf
cat << EOF > initramfs/etc/nsswitch.conf
passwd:		files
group:		files
shadow:		files
hosts:		files
networks:	files
protocols:	files
services:	files
ethers:		files
rpc:		files
netgroup:	files
EOF

```
添加基本组:    

```
touch initramfs/etc/group
cat << EOF > initramfs/etc/group
root:x:0:
daemon:x:1:
tty:x:5:
disk:x:6:
lp:x:7:
kmem:x:15:
dialout:x:20:
cdrom:x:24:
tape:x:26:
audio:x:29:
video:x:44:
input:x:122:
EOF
```
创建启动脚本，下面的脚本负责挂载特定目录，启动udev， 最后拉起petitboot, 也可以在此基础上更改为你自己的应用程序:      

```
touch initramfs/init
cat << EOF > initramfs/init
#!/bin/sh

/bin/busybox --install -s

CURRENT_TIMESTAMP=\$(date '+%s')
if [ \$CURRENT_TIMESTAMP -lt `date '+%s'` ]; then
	date -s "@`date '+%s'`"
fi

mount -t proc proc /proc
mount -t sysfs sysfs /sys
mount -t devtmpfs none /dev

echo 0 > /proc/sys/kernel/printk
clear

systemd-udevd &
udevadm hwdb --update
udevadm trigger

pb-discover &
petitboot-nc

if [ -e /etc/pb-lockdown ]; then
	echo "Failed to launch petitboot, rebooting!"
	echo 1 > /proc/sys/kernel/sysrq
	echo b > /proc/sysrq-trigger
else
	echo "Failed to launch petitboot, dropping to a shell"
	exec sh
fi
EOF
chmod +x initramfs/init
```
去掉调试符号:   

```
strip initramfs/sbin/*
strip initramfs/usr/sbin/*
strip initramfs/lib/x86_64-linux-gnu/*
strip initramfs/usr/lib/x86_64-linux-gnu/*
strip initramfs/usr/lib/udev/*_id
```
创建CPIO并压缩镜像:     

```
cd initramfs
find . | cpio -H newc -o > ../initramfs.cpio
cd ..
cat initramfs.cpio | lzma > initramfs.igz
```
编译内核:    

```
git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
git reset --hard 2dcd0af568b0cf583645c8a317dd12e344b1c72a
make menuconfig
make -j`nproc` bzImage
```
内核配置项中需要调整的部分:    

```
Processor type and features  --->
	[*] kexec file based system call
	[ ]   Verify kernel signature during kexec_file_load() syscall

Device Drivers  --->
	Generic Driver Options  --->
		[ ]   Include in-kernel firmware blobs in kernel binary
	HID support  --->
		{*} HID bus support
		<*>   Generic HID driver
			USB HID support  --->
			<*> USB HID transport layer
	[*] USB support  --->
		<*>     xHCI HCD (USB 3.0) support
		{*}       Generic xHCI driver for a platform device
		<*>     EHCI HCD (USB 2.0) support
		<*>     OHCI HCD (USB 1.1) support
		<*>       OHCI support for PCI-bus USB controllers
		{*}       Generic OHCI driver for a platform device
		<*>     UHCI HCD (most Intel and VIA) support
		<*>     USB Mass Storage support
			<Enable all options in this category as kernel builtins except verbose debug>

Kernel hacking  --->
	Compile-time checks and compiler options  --->
		[ ] Compile the kernel with debug info
	[ ] KGDB: kernel debugger  ----
	[ ] Enable verbose x86 bootup info messages
	[ ] Early printk
		[ ]   Early printk via EHCI debug port
		[ ]   Early printk via the EFI framebuffer

File systems  --->
	-*- Native language support  --->
		
General setup  --->
	Compiler optimization level (Optimize for size)  --->
```
coreboot下，调整配置:     

```
make menuconfig


General setup  --->
	[ ] Build the ramstage to be relocatable in 32-bit address space.

Mainboard  --->
	ROM chip size (16384 KB (16 MB))  --->
	(0x1000000) Size of CBFS filesystem in ROM

Payload  --->
	Add a payload (A Linux payload)  --->
		(X) A Linux payload
	Linux path and filename
		../../linux/linux/arch/x86_64/boot/bzImage
	Linux initrd
		../../initramfs.igz
	Linux command line
		console=ttyS0,115200n8 console=tty0 panic=60 softlockup_panic=60 nmi_watchdog=1 quiet rw
```
这里我换成了q35:    

![/images/20240913_111649_x.jpg](/images/20240913_111649_x.jpg)

编译， 而后启动:    

```
make -j12
ls build/coreboot.rom -l -h
qemu-system-x86_64 -m 1G -M q35 -serial stdio -bios coreboot/coreboot/build/coreboot.rom
```
一个开启了ssh/vnc的命令行:     

```
qemu-system-x86_64 -m 1G -M pc -boot d -cdrom ./ubuntu-18.04.6-server-amd64.iso -hda ./zzzz_1604.qcow2 -serial stdio -bios coreboot.rom  -net nic -net user,hostfwd=tcp::2288-:22 -vga std -vnc :7

```
目前问题： q35启动有问题，i440无法使用光驱，图形无法使用等.     
