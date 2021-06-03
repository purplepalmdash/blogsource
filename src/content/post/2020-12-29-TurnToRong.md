+++
title= "TurnToRong"
date = "2020-12-29T12:15:36+08:00"
description = "TurnToRong"
keywords = ["Technology"]
categories = ["Technology"]
+++
现场安装时，因为某些不可控的原因，可能无法安装定制化操作系统，此时可使用以下步骤，从最小化安装的Ubuntu18.04 转换为RONG节点:    

以下操作以`Ubuntu18.04.5`为例说明，默认操作用户为安装时创建的用户`kkk`, 现场需要根据情况灵活调整。

1. 上传ISO到机器:   

```
# scp ./ubuntu-18.04.5-server-amd64-auto-xfs.iso kkk@192.168.122.32:/home/kkk
kkk@192.168.122.32's password: 
```
2. 在机器上挂载iso:    

```
kkk@ubuntu:~$ sudo mount ubuntu-18.04.5-server-amd64-auto-xfs.iso /media/cdrom
[sudo] password for kkk: 
mount: /mnt: WARNING: device write-protected, mounted read-only.
```

3. 使用iso作为本地安装源:    

```
# rm -f /etc/apt/sources.list
# apt-cdrom -m -d=/media/cdrom add
# cat /etc/apt/sources.list
deb cdrom:[Ubuntu-Server 18.04.5 LTS _Bionic Beaver_ - Release amd64 (20200810)]/ bionic main restricted

```
4. 此时`apt-get`更新源并安装对应的包:   

```
# apt-get update 
# apt-get install nfs-common openssh-server update-motd parted build-essential telnet tcpdump python
```
安装完毕后程序会自动umount `/media/cdrom`下挂载的ISO， 如果提示需要重新mount `/media/cdrom`的时候，则在另一终端重新mount iso至`/media/cdrom`下则可。

5. 注入root免登录密钥

```
$ sudo su
#  ssh-keygen 
一路按回车，创建公钥私钥
# vim  /root/.ssh/authorized_keys
粘贴以下内容, 此内容在rong ISO的preseed/auto.seed中可以找到, 开头为"ssh-rsa", 结尾为"DashSSD"标识.

ssh-rsa owaugowugouwoguwougowuoguwougouwogwe例子例子例子例子例子例子**************= dash@DashSSD
```

6. 此时可以进行RONG的正常部署, 不一定需要使用test用户登录。
