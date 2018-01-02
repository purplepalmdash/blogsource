---
categories: ["Virtualization"]
comments: true
date: 2015-11-16T21:20:00Z
title: Spice Client
url: /2015/11/16/spice-client/
---

### Image Conversion
Convert the vdi files into qcow2 file:    

```
$ qemu-img convert -f vdi -O qcow2 Windows81.vdi Windows81.qcow2
```

Then continue to create the virtual machine via importing the img.    

### Spice Client
Using virtviewer for view the remote machine.   

```
$ sudo pacman -S virtviewer
$ remote-viewer spice://localhost:5900
```
Or you could view the desktop via `spicec`.   

The listening port could be view via `netstat -anp | grep 5900`.       
