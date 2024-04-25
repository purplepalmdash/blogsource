+++
title= "safebootloaderTips"
date = "2024-04-25T09:27:59+08:00"
description = "safebootloaderTips"
keywords = ["Technology"]
categories = ["Technology"]
+++
From Makefile:    

```
O ?= ./build
......
$O/bootx64.efi: $O/chainload/loader.efi $O/vmlinuz $O/initrd.cpio.xz
	$O/chainload/unify-kernel $@ \
		linux=$O/vmlinuz \
		initrd=$O/initrd.cpio.xz \
		cmdline=config/cmdline-5.4.117.txt
```
file content:    

```
kkk@kkk:~/safeboot-loader$ ls build/chainload/loader.efi 
build/chainload/loader.efi
kkk@kkk:~/safeboot-loader$ file build/chainload/loader.efi 
build/chainload/loader.efi: PE32+ executable (EFI application) x86-64 (stripped to external PDB), for MS Windows
kkk@kkk:~/safeboot-loader$ ls build/chainload/loader.efi  -l -h
-rwxrwxr-x 1 idv idv 52K  4月 18 14:32 build/chainload/loader.efi
kkk@kkk:~/safeboot-loader$ vim build/chainload/loader.efi 
kkk@kkk:~/safeboot-loader$ ls build/vmlinuz 
build/vmlinuz
kkk@kkk:~/safeboot-loader$ ls build/vmlinuz  -l -h
-rw-rw-r-- 1 idv idv 2.5M  4月 18 10:12 build/vmlinuz
kkk@kkk:~/safeboot-loader$ ls build/initrd.cpio.xz -l -h
-rw-rw-r-- 1 idv idv 13M  4月 18 14:32 build/initrd.cpio.xz
kkk@kkk:~/safeboot-loader$ ls config/cmdline-5.4.117.txt 
config/cmdline-5.4.117.txt
kkk@kkk:~/safeboot-loader$ cat config/cmdline-5.4.117.txt 
earlyprintk=serial,ttyS0,115200 console=tty0 console=ttyS0,115200 noefi acpi=of
```
