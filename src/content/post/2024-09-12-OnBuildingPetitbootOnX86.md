+++
title= "OnBuildingPetitbootOnX86"
date = "2024-09-12T22:30:09+08:00"
description = "OnBuildingPetitbootOnX86"
keywords = ["Technology"]
categories = ["Technology"]
+++
Building coreboot Steps(half):   
```
 sudo apt install -y build-essential git vim
 mkdir Code
 cd Code
 mkdir coreboot
 cd coreboot/
 git clone http://review.coreboot.org/p/coreboot
 cd coreboot/
 git checkout tags/4.6 -b local46
 sudo apt install -y m4 bison flex zlib1g-dev libncurses5-dev
 wget https://fossies.org/linux/misc/old/libelf-0.8.13.tar.gz
 mv libelf-0.8.13.tar.gz util/crossgcc/tarballs/
 history
 vim util/crossgcc/buildgcc 
IASL_ARCHIVE="https://downloadmirror.intel.com/774735/acpica-unix2-${IASL_VERSION}.tar.gz"
 make crossgcc CPUS=`nproc`
 make iasl CPUS=`nproc`
 make menuconfig
```

Building systemd:   

``` 
cd ~/Code
 mkdir systemd
 cd systemd/
 wget https://github.com/systemd/systemd/archive/refs/tags/v229.tar.gz
 tar xzvf v229.tar.gz 
 mv systemd-229/ systemd
 cd systemd/
 sudo apt install intltool
 sudo apt install -y libtool
 sudo apt install -y gperf
 sudo apt install libcap-dev
 sudo apt install -y libblkid-dev
 sudo apt install -y libmount-dev
 sudo apt install xsltproc
 sudo apt install docbook-xsl
 mkdir build
 cd build
 ../configure --prefix=/usr --enable-blkid --disable-seccomp --disable-libcurl --disable-pam --disable-kmod
 make -j12
```
Building kexec:    

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
Building twin:    

```
mkdir petitboot
cd petitboot/
git clone git://git.kernel.org/pub/scm/linux/kernel/git/geoff/libtwin.git
cd libtwin/
./autogen.sh 
cp README.md README
./autogen.sh 
make -j8
sudo make install
cd ../../
```
Building petitboot:    

```
cd petitboot/
wget https://git.raptorengineering.com/git/petitboot/snapshot/petitboot-1.4.3.tar.gz
tar xzvf petitboot-1.4.3.tar.gz 
mv petitboot-1.4.3/ petitboot
cd petitboot/
./bootstrap 
sudo apt install -y autopoint
sudo apt install -y libgpgme11-dev
sudo apt install -y libdevmapper-dev
./bootstrap 
CPPFLAGS="-I../../systemd/systemd/src/libudev/" LDFLAGS="-L../../systemd/systemd/build/.libs/" ./configure --prefix=/usr --enable-static --disable-shared --enable-busybox --with-ncurses --without-twin-x11 --without-twin-fbdev --with-signed-boot
make -j12
cd ../../
```
Building busybox:    

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
![/images/2024_09_12_22_59_00_740x804.jpg](/images/2024_09_12_22_59_00_740x804.jpg)

![/images/2024_09_12_22_59_29_438x174.jpg](/images/2024_09_12_22_59_29_438x174.jpg)

Install dw packages:   

```
sudo apt install -y libdw-dev libdw1
```


```
Initramfs Build
Now that the helper applications have been built, a minimal initramfs can be assembled.
The following commands assume you are building the firmware image on a 64-bit x86 system for a 64-bit x86 target. Please replace "x86_64-linux-gnu" with the correct architecture tuple as needed. As an example, on a ppc64el system the "powerpc64le-linux-gnu" tuple would be used instead.

Prepare the skeleton directory structure

mkdir initramfs
mkdir -p initramfs/{bin,sbin,etc,lib,proc,sys,newroot,usr,usr/bin,usr/sbin,var,var/log,run,run/udev,tmp}
mkdir initramfs/var/log/petitboot
touch initramfs/etc/mdev.conf
cp -Rp /lib/terminfo initramfs/lib/
cp -Rp busybox/busybox/busybox initramfs/bin/
ln -s busybox initramfs/bin/sh
Copy core libraries to the new initramfs

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
Copy helper binaries to the new initramfs

cp -Rp /usr/bin/gpg initramfs/usr/bin/

cp systemd/systemd/build/.libs/libudev.so.* initramfs/lib/x86_64-linux-gnu/
cp -Rp systemd/systemd/build/systemd-udevd initramfs/sbin/
cp -Rp systemd/systemd/build/udevadm initramfs/sbin/

mkdir -p initramfs/usr/lib/udev
cp -Rp systemd/systemd/build/*_id initramfs/usr/lib/udev

cp -Rp kexec/kexec-tools/build/sbin/kexec initramfs/sbin/
Install petitboot itself to the initramfs

cd petitboot/petitboot
make DESTDIR=`realpath ../../initramfs/` install
cd ../..
Copy udev rules to the new initramfs

mkdir -p initramfs/usr/lib/udev/rules.d
cp -Rp systemd/systemd/rules/* initramfs/usr/lib/udev/rules.d/
cp -Rp systemd/systemd/build/rules/* initramfs/usr/lib/udev/rules.d/
rm -f initramfs/usr/lib/udev/rules.d/*-drivers.rules
Set up udhcp helper scripts

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
Set up nsswitch

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
Add basic groups

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
Create boot script

The following script is automatically used on every system start to mount needed special directories, start udev, and finally launch petitboot. It can be customized as required for your particular application.

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
OPTIONAL: Set up GPG keyring for signed or encrypted boot

If you are setting up GPG signature checking or encryption, you will need to export the public key of your GPG kernel signer account, in ASCII format, to the file "public_key.txt". For encrypted kernels, you will also need to export the machine's private key in ASCII format to the file "private_key.txt". The machine private key is a specific, dedicated GPG account that should be created only for a single machine; encrypted kernels will use this GPG account as the recipient and the kernel signer as the source GPG account. Both files should be placed in the petitboot build root directory, and the private key (if present) should be chmod 600 (read/write by owner only).

WARNING: DO NOT export your personal private key, the private key of the kernel signer, or any other private keys from your GPG keyring! They are not needed by the build process.
mkdir initramfs/etc/gpg
gpg --homedir=initramfs/etc/gpg --import public_key.txt
gpg --homedir=initramfs/etc/gpg --import private_key.txt
echo "`gpg --homedir=initramfs/etc/gpg --fingerprint | grep "Key fingerprint" | sed 's/.*Key fingerprint = //g' \
	| sed 's/ //g'`:6:" | gpg --homedir=initramfs/etc/gpg --import-ownertrust
chown -R root initramfs/etc/gpg
chgrp -R root initramfs/etc/gpg
chmod -R 400 initramfs/etc/gpg
To only boot signed kernels, execute the following commands:

echo "`gpg --homedir=initramfs/etc/gpg --fingerprint | grep "Key fingerprint" | sed 's/.*Key fingerprint = //g' \
	| sed 's/ //g'`" >> initramfs/etc/pb-lockdown
To only boot kernels that have been both encrypted and signed, execute the following commands:

echo "ENCRYPTED" > initramfs/etc/pb-lockdown
echo "`gpg --homedir=initramfs/etc/gpg --fingerprint | grep "Key fingerprint" | sed 's/.*Key fingerprint = //g' \
	| sed 's/ //g'`" >> initramfs/etc/pb-lockdown
Strip debug symbols from files installed in the initramfs

This step is crucial to reduce the initramfs size down to a range that will fit on a typical Flash ROM. Leaving unstripped binaries with debug symbols intact can more than double the size of the compressed initramfs!

strip initramfs/sbin/*
strip initramfs/usr/sbin/*
strip initramfs/lib/x86_64-linux-gnu/*
strip initramfs/usr/lib/x86_64-linux-gnu/*
strip initramfs/usr/lib/udev/*_id
CPIO creation and image compression

cd initramfs
find . | cpio -H newc -o > ../initramfs.cpio
cd ..
cat initramfs.cpio | lzma > initramfs.igz
```
Building kernel:    

```
git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
cd linux
git reset --hard 2dcd0af568b0cf583645c8a317dd12e344b1c72a
make menuconfig
sudo apt install libssl-dev
make -j`nproc` bzImage
```
Menuconfig:    

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
### coreboot(second stage)
Now we could build:    

```
make -j12
```
Verify the result:    

```
ls build/coreboot.rom -l -h
-rw-rw-r-- 1 dash dash 16M 9月  12 23:35 build/coreboot.rom

```

Verification:    

```
qemu-system-x86_64 -m 1G -M q35 -serial stdio -bios coreboot/coreboot/build/coreboot.rom

```

![/images/2024_09_12_23_39_50_715x430.jpg](/images/2024_09_12_23_39_50_715x430.jpg)

