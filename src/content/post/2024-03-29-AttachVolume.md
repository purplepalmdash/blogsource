+++
title= "AttachVolume"
date = "2024-03-29T10:19:02+08:00"
description = "AttachVolume"
keywords = ["Technology"]
categories = ["Technology"]
+++
创建一个名为`charlie`的容器:     

```
$ sudo docker run --name charlie -ti ubuntu bash
```
运行以下命令，安装`docker-enter`等一系列工具到`/usr/local/bin/`下:     

```
$ sudo docker run --rm -v /usr/local/bin:/target jpetazzo/nsenter
```
创建一个名为`attach.sh`的脚本, 这里为了简单起见，写死了`CONTAINER`及`HOSTPATH`/`CONPATH`等:      

```
#!/bin/sh
set -e
CONTAINER=charlie
HOSTPATH=/home/dash/Work/DOCKER/docker
CONTPATH=/src

REALPATH=$(readlink --canonicalize $HOSTPATH)
FILESYS=$(df -P $REALPATH | tail -n 1 | awk '{print $6}')

while read DEV MOUNT JUNK
do [ $MOUNT = $FILESYS ] && break
done </proc/mounts
[ $MOUNT = $FILESYS ] # Sanity check!

while read A B C SUBROOT MOUNT JUNK
do [ $MOUNT = $FILESYS ] && break
done < /proc/self/mountinfo 
[ $MOUNT = $FILESYS ] # Moar sanity check!

SUBPATH=$(echo $REALPATH | sed s,^$FILESYS,,)
DEVDEC=$(printf "%d %d" $(stat --format "0x%t 0x%T" $DEV))
echo "1"
docker-enter $CONTAINER  sh -c \
	     "[ -b $DEV ] || mknod --mode 0600 $DEV b $DEVDEC"
echo "2"
docker-enter $CONTAINER  mkdir /tmpmnt
docker-enter $CONTAINER  mount $DEV /tmpmnt
docker-enter $CONTAINER  mkdir -p $CONTPATH
docker-enter $CONTAINER  mount -o bind /tmpmnt/$SUBROOT/$SUBPATH $CONTPATH
docker-enter $CONTAINER  umount /tmpmnt
docker-enter $CONTAINER  rmdir /tmpmnt
```
使用方法:    

```
### 这里对应到脚本中的HOSTPATH字段，映射主机目录
$ mkdir -p /home/dash/Work/DOCKER/docker
$ touch /home/dash/Work/DOCKER/docker/ccc
$ ./attach.sh
```
进入到容器中检查目录映射, 可以看到我们touch出来的`ccc`文件已在映射后的容器目录中：   

```
$ ls /src/
ccc
```
