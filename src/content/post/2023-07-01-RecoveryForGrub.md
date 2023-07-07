+++
title= "RecoveryForGrub"
date = "2023-07-01T08:52:38+08:00"
description = "RecoveryForGrub"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 目的
GRUB（Grand Unified Bootloader）是一个用于引导和加载操作系统内核的工具，也是基于Linux内核的系统的默认引导加载程序。尽管它在机器打开时首先运行，但普通用户很少看到 GRUB 运行。它自动运行，不需要用户输入。    
然而，当尝试在同一台机器上引导另一个操作系统与 Linux并存时，另一个系统的引导加载程序可能会覆盖 GRUB，从而导致 Linux 系统无法引导。    
本文将介绍如何使用 GRUB Rescue 命令和引导修复工具修复 Linux 引导失败。       
### 先决条件
1. 具有 sudo 权限的帐户。
2. 访问命令行。

### Grub启动问题
GRUB 无法引导到操作系统的最常见原因是另一个操作系统的引导加载程序覆盖了 GRUB 引导配置。在尝试与现有 Linux 安装进行双重引导时会出现此问题。另一个原因是意外删除了 GRUB 配置文件。    
当 GRUB 无法引导系统时，会出现 GRUB Rescue 提示符。    

![/images/2023_07_01_08_54_21_274x68.jpg](/images/2023_07_01_08_54_21_274x68.jpg)

上面的示例显示 GRUB 在显示 grub 救援提示之前显示“无此类分区”错误。另一个常见的 GRUB 错误是“未知文件系统”，后面跟着相同的提示。    

![/images/2023_07_01_08_54_52_257x58.jpg](/images/2023_07_01_08_54_52_257x58.jpg)

有时候将直接显示grub提示符:    

![/images/2023_07_01_08_55_09_160x37.jpg](/images/2023_07_01_08_55_09_160x37.jpg)

### Grub恢复命令
以下是常用 GRUB Rescue 命令的列表。使用上一节中提到的提示中的命令。   

```
Command	Description	Example
boot	Start booting (shortcuts: F10, CTRL + x).	The command is issued without arguments.
cat	Write the contents of a file to standard output.	cat (hd0,1)/boot/grub/grub.cfg
configfile	Load a configuration file.	configfile (hd0,1)/boot/grub/grub.cfg
initrd	Load the initrd.img file.	initrd (hd0,1)/initrd.img
insmod	Load a module.	insmod (hd0,1)/boot/grub/normal.mod
loopback	Mount an image file as a device.	loopback loop0 (hd0,1)/iso/image.iso
ls	Display the contents of a directory or partition.	ls (hd0,1)
lsmod	Display a list of loaded modules.	The command is issued without arguments.
normal	Activate the normal module.	The command is issued without arguments.
search	Search for devices. Option --file searches for files, --label searches for labels, --fs-uuid searches for filesystem UUID.	search -file [filename]
set	Set an environment variable. If issued with no arguments, the command prints the list of all environment variables and their values.	set [variable-name]=[value]
```
### 修复启动问题
本教程介绍了解决 GRUB 引导问题的两种方法：使用 GRUB Rescue 提示符和引导修复工具。    
#### 使用GRUB Rescue提示符
1. 使用不带参数的 set 命令查看环境变量：    

```
set
```
这个例子的输出显示，GRUB被设置为从（hd0,msdos3）分区启动：    

![/images/2023_07_01_08_57_34_428x263.jpg](/images/2023_07_01_08_57_34_428x263.jpg)

2. ls命令列出了磁盘上的可用分区。    

```
ls
```
输出展示了分区列表:    

![/images/2023_07_01_08_58_17_510x76.jpg](/images/2023_07_01_08_58_17_510x76.jpg)

使用ls命令找到包含启动目录的分区。    

```
ls [partition-name]
```
这个例子显示了(hd0,msdos1)分区中的启动目录。    

![/images/2023_07_01_08_58_50_744x95.jpg](/images/2023_07_01_08_58_50_744x95.jpg)

3. 将引导分区设置为根变量的值。该示例使用名为 (hd0,msdos1) 的分区。   

```
set root=(hd0,msdos1)
```
4. 加载normal启动模式:    

```
insmod normal
```
5. 启动normal启动模式:    

```
normal
```
normal启动模式下允许你能使用更复杂的命令修复启动    

6. 使用linux命令加载Linux内核:    

```
linux /boot/vmlinuz-4.19.12-generic root=/dev/sda1 ro
```
7. 调用boot命令:    

```
boot
```
现在系统应该可以正常启动了。    

这篇文章了讲述了在命令行下使用grub rescue修复系统启动的话题，下一部分将讲述如何在使用Livecd启动到图形界面后执行grub rescue的步骤。
