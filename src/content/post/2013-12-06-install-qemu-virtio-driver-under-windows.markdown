---
categories: ["qemu", "Linux"]
comments: true
date: 2013-12-06T00:00:00Z
title: Install Qemu virtio driver under windows
url: /2013/12/06/install-qemu-virtio-driver-under-windows/
---

Download the iso file from the redhat repository:    
[http://alt.fedoraproject.org/pub/alt/virtio-win/latest/images/images/images/bin/src/](http://alt.fedoraproject.org/pub/alt/virtio-win/latest/images/images/images/bin/src/)    
Start the qemu with the following command :      
```
 	./run-qemu -hda fpgawindows.qcow2 -m 1024 -cdrom ./virtio-win-0.1-74.iso -drive file=./fake.qcow2,if=ide
```
In run-qemu, the actual command is:     
```
	qemu-system-i386 -net nic,model=virtio,macaddr=$macaddr -net tap,ifname="$IFACE" $*
```
Then, follow the following images to operate: 

![qemu1.jpg](/images/qemu1.jpg)    

![qemu2.jpg](/images/qemu2.jpg)    

![qemu3.jpg](/images/qemu3.jpg)    

![qemu4.jpg](/images/qemu4.jpg)    

![qemu5.jpg](/images/qemu5.jpg)
