+++
title= "SG1OnArchLinux"
date = "2022-08-15T14:21:06+08:00"
description = "SG1OnArchLinux"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Steps
Install sddm:    

```
# pacman -S sddm xorg
```
After reboot, examine the kernel and the sg1 pci infos:    

```
[root@archvfio ~]# lspci | grep -i vga
00:01.0 VGA compatible controller: Red Hat, Inc. Virtio GPU (rev 01)
07:00.0 VGA compatible controller: Intel Corporation SG1 [Server GPU SG-18M] (rev 01)
[root@archvfio ~]# uname -a
Linux archvfio 5.19.1-arch2-1 #1 SMP PREEMPT_DYNAMIC Thu, 11 Aug 2022 16:06:13 +0000 x86_64 GNU/Linux
```
Add virtio gpu into blacklist:    

```
# vim /etc/modprobe.d/blacklist.conf
   # blacklist virtio_gpu
```
Didn't take effects, change the graphical to qxl and added blacklist qxl:    

![/images/2022_08_15_14_23_35_535x518.jpg](/images/2022_08_15_14_23_35_535x518.jpg)

Take effects:     

```
[root@archvfio ~]# lsmod | grep qxl
[root@archvfio ~]# ls /dev/dri/
by-path  card0	renderD128
```
Examine the sg1 connected info:    

```
for p in /sys/class/drm/*/status; do con=${p%/status}; echo -n "${con#*/card?-}: "; cat $p; done
DP-1: disconnected
DP-2: disconnected
DP-3: disconnected
HDMI-A-1: disconnected
HDMI-A-2: disconnected
HDMI-A-3: disconnected
HDMI-A-4: disconnected
```
Edit the Xorg configuration files and start the benchmark:    

```
# cat /etc/X11/xorgintel.conf 
Section "Device"
    Identifier     "Device[0]"
    BusID          "PCI:7:0:0"
    VendorName     "Intel"
    BoardName      "DG1"
    Option "AllowEmptyInitialConfiguration"
EndSection
Section "ServerFlags"
    Option "Debug" "dmabuf_capable"
EndSection
Section "Monitor"
    Identifier   "Monitor0"
    VendorName   "Monitor Vendor"
    ModelName    "Monitor Model"
    Option   "IgnoreEDID"
EndSection
# Xorg :179 -config /etc/X11/xorgintel.conf
# ps -ef | grep -i xorg
root         841     526  0 14:27 tty2     00:00:00 /usr/lib/Xorg :179 -config /etc/X11/xorgintel.conf
root         877     873  0 14:28 pts/1    00:00:00 grep -i xorg

```
### testing
Install x11vnc and glmark2 for  benchmarking:   

```
# vim /etc/pacman.conf
[archlinuxcn]
#The Chinese Arch Linux communities packages.
SigLevel = Never
Server   = http://repo.archlinuxcn.org/$arch
# pacman -Sy
# pacman -S x11vnc glmark2
[root@archvfio ~]# export DISPLAY=:179
[root@archvfio ~]# glmark2
=======================================================
    glmark2 2021.12
=======================================================
    OpenGL Information
    GL_VENDOR:     Intel
    GL_RENDERER:   Mesa Intel(R) Graphics (SG1)
    GL_VERSION:    4.6 (Compatibility Profile) Mesa 22.1.6
=======================================================

```
glmark2 result:    


```
=======================================================
    glmark2 2021.12
=======================================================
    OpenGL Information
    GL_VENDOR:     Intel
    GL_RENDERER:   Mesa Intel(R) Graphics (SG1)
    GL_VERSION:    4.6 (Compatibility Profile) Mesa 22.1.6
=======================================================
[build] use-vbo=false: FPS: 2386 FrameTime: 0.419 ms
[build] use-vbo=true: FPS: 3112 FrameTime: 0.321 ms
[texture] texture-filter=nearest: FPS: 3262 FrameTime: 0.307 ms
[texture] texture-filter=linear: FPS: 3452 FrameTime: 0.290 ms
[texture] texture-filter=mipmap: FPS: 3493 FrameTime: 0.286 ms
[shading] shading=gouraud: FPS: 3056 FrameTime: 0.327 ms
[shading] shading=blinn-phong-inf: FPS: 3070 FrameTime: 0.326 ms
[shading] shading=phong: FPS: 2990 FrameTime: 0.334 ms
[shading] shading=cel: FPS: 3129 FrameTime: 0.320 ms
[bump] bump-render=high-poly: FPS: 2409 FrameTime: 0.415 ms
[bump] bump-render=normals: FPS: 3495 FrameTime: 0.286 ms
[bump] bump-render=height: FPS: 3393 FrameTime: 0.295 ms
[effect2d] kernel=0,1,0;1,-4,1;0,1,0;: FPS: 2885 FrameTime: 0.347 ms
[effect2d] kernel=1,1,1,1,1;1,1,1,1,1;1,1,1,1,1;: FPS: 2300 FrameTime: 0.435 ms
[pulsar] light=false:quads=5:texture=false: FPS: 3424 FrameTime: 0.292 ms
[desktop] blur-radius=5:effect=blur:passes=1:separable=true:windows=4: FPS: 1861 FrameTime: 0.537 ms
[desktop] effect=shadow:windows=4: FPS: 2431 FrameTime: 0.411 ms
[buffer] columns=200:interleave=false:update-dispersion=0.9:update-fraction=0.5:update-method=map: FPS: 769 FrameTime: 1.300 ms
[buffer] columns=200:interleave=false:update-dispersion=0.9:update-fraction=0.5:update-method=subdata: FPS: 1275 FrameTime: 0.784 ms
[buffer] columns=200:interleave=true:update-dispersion=0.9:update-fraction=0.5:update-method=map: FPS: 824 FrameTime: 1.214 ms
[ideas] speed=duration: FPS: 3128 FrameTime: 0.320 ms
[jellyfish] <default>: FPS: 2708 FrameTime: 0.369 ms
[terrain] <default>: FPS: 509 FrameTime: 1.965 ms
[shadow] <default>: FPS: 2664 FrameTime: 0.375 ms
[refract] <default>: FPS: 1041 FrameTime: 0.961 ms
[conditionals] fragment-steps=0:vertex-steps=0: FPS: 3226 FrameTime: 0.310 ms
[conditionals] fragment-steps=5:vertex-steps=0: FPS: 3035 FrameTime: 0.329 ms
[conditionals] fragment-steps=0:vertex-steps=5: FPS: 2974 FrameTime: 0.336 ms
[function] fragment-complexity=low:fragment-steps=5: FPS: 3014 FrameTime: 0.332 ms
[function] fragment-complexity=medium:fragment-steps=5: FPS: 3047 FrameTime: 0.328 ms
[loop] fragment-loop=false:fragment-steps=5:vertex-steps=5: FPS: 3250 FrameTime: 0.308 ms
[loop] fragment-steps=5:fragment-uniform=false:vertex-steps=5: FPS: 3050 FrameTime: 0.328 ms
[loop] fragment-steps=5:fragment-uniform=true:vertex-steps=5: FPS: 3293 FrameTime: 0.304 ms
=======================================================
                                  glmark2 Score: 2665 
=======================================================
```
Unigen valley benchmark:  

```
# export DISPLAY=:179
[root@archvfio ~]# x11vnc
###############################################################
#@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@#
#@                                                           @#
#@  **  WARNING  **  WARNING  **  WARNING  **  WARNING  **   @#
#@                                                           @#
#@        YOU ARE RUNNING X11VNC WITHOUT A PASSWORD!!        @#
#@                                                           @#
#@  This means anyone with network access to this computer   @#


On another terminal:  
[root@archvfio ~]# export DISPLAY=:179
[root@archvfio ~]# cd Unigine_Valley-1.0/
[root@archvfio Unigine_Valley-1.0]# ls
bin  data  documentation  valley
[root@archvfio Unigine_Valley-1.0]# ./valley 
``` 

![/images/2022_08_15_14_42_10_965x786.jpg](/images/2022_08_15_14_42_10_965x786.jpg)

valley cannot start in default resolution, changes edid and set the default resolution to 1920x1080:    

```
# vim /etc/default/grub
........
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet drm_kms_helper.edid_firmware=edid/1920x1080.bin video=HDMI-A-1:e"
........
# grub-mkconfig -o /boot/grub/grub.cfg
```
Valley result:    

![/images/2022_08_15_14_57_02_518x465.jpg](/images/2022_08_15_14_57_02_518x465.jpg)

