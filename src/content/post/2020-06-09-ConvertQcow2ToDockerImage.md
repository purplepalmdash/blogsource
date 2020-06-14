+++
title= "ConvertQcow2ToDockerImage"
date = "2020-06-09T11:31:31+08:00"
description = "ConvertQcow2ToDockerImage"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Create vm
Create vm disks:    

```
# qemu-img create -f qcow2 2004.qcow2 6G
```
Install system into this qcow2 files with your customized settings.   

### Convert
Convert into img file:   

```
$ qemu-img convert -f vmdk -O raw 2004.qcow2 2004.img
```
Using guestfish for converting into docker images:    

```
$ sudo guestfish -a 2004.img --ro
$ ><fs> run
$ ><fs> list-filesystems
/dev/sda1: ext4
/dev/sda2: unknown
/dev/sda5: swap
$ ><fs> mount /dev/sda1 /
$ ><fs> tar-out / - | xz --best >> my2004.xz
$ ><fs> exit
$ cat my2004.xz | docker import - YourImagesName
```
Now push it into dockerhub, next time you could use it freely.   
