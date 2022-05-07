+++
title= "TipsForAOSPGPU"
date = "2022-05-07T10:39:19+08:00"
description = "TipsForAOSPGPU"
keywords = ["Technology"]
categories = ["Technology"]
+++
Translation mode:    

```
# emulator -gpu host -qemu -cpu host

In adb shell :  
# dumpsys | grep -i opengl                                                                                                                                                                       
GLES: Google (AMD), Android Emulator OpenGL ES Translator (AMD Radeon RX 5700 (navi10, LLVM 14.0.0, DRM 3.35, 5.4.0-107-generic)), OpenGL ES 2.0 (4.6 (Core Profile) Mesa 22.2.0-devel (git-5e84335 2022-04-22 focal-oibaf-ppa))

generic_x86_64:/ # ls -l -h /dev/graphics/fb0       
crw-rw---- 1 root graphics 29,   0 2022-05-07 10:37 /dev/graphics/fb0

```
更改为virtio级别:    

emulator感觉比较老了，需要换成cuttlefish来试验
