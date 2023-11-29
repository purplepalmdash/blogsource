+++
title= "TipsOnIDVVDIFusion"
date = "2023-11-29T15:05:19+08:00"
description = "TipsOnIDVVDIFusion"
keywords = ["Technology"]
categories = ["Technology"]
+++
Modification steps:    

Rebuild qemu:       

```
sudo apt install -y librbd-dev
cd qemu-7.1.0/
./configure --target-list=x86_64-softmmu --enable-debug --disable-docs --disable-virglrenderer --prefix=/usr --enable-virtfs --enable-libusb --disable-debug-tcg --audio-drv-list=pa,alsa --enable-spice --enable-rbd
make -j8 && make install
```
