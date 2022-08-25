+++
title= "SG1OnArchLinux2"
date = "2022-08-25T08:37:48+08:00"
description = "SG1OnArchLinux2"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Installation
After install arch system, install packages:    

```
# pacman -S gnome libva-mesa-driver  mesa-demos mesa-utils mesa-vdpau opencl-mesa vulkan-intel vulkan-mesa-layers vulkan-radeon adriconf awesome x11vnc net-tools
```
List all of the sg socs:    

```
# lspci | grep -i vga
0000:05:00.0 VGA compatible controller: ASPEED Technology, Inc. ASPEED Graphics Family (rev 41)
0000:b3:00.0 VGA compatible controller: Intel Corporation SG1 [Server GPU SG-18M] (rev 01)
0000:b8:00.0 VGA compatible controller: Intel Corporation SG1 [Server GPU SG-18M] (rev 01)
0000:bd:00.0 VGA compatible controller: Intel Corporation SG1 [Server GPU SG-18M] (rev 01)
0000:c2:00.0 VGA compatible controller: Intel Corporation SG1 [Server GPU SG-18M] (rev 01)
```
List all of the render nodes:    

```
# ls /dev/dri/
by-path  card0	card1  card2  card3  card4  renderD128	renderD129  renderD130	renderD131
# ls /dev/dri/by-path/ -l
lrwxrwxrwx 1 root root  8 Aug 25 08:30 pci-0000:05:00.0-card -> ../card0
lrwxrwxrwx 1 root root  8 Aug 25 08:30 pci-0000:b3:00.0-card -> ../card1
lrwxrwxrwx 1 root root 13 Aug 25 08:30 pci-0000:b3:00.0-render -> ../renderD128
lrwxrwxrwx 1 root root  8 Aug 25 08:30 pci-0000:b8:00.0-card -> ../card2
lrwxrwxrwx 1 root root 13 Aug 25 08:30 pci-0000:b8:00.0-render -> ../renderD129
lrwxrwxrwx 1 root root  8 Aug 25 08:30 pci-0000:bd:00.0-card -> ../card3
lrwxrwxrwx 1 root root 13 Aug 25 08:30 pci-0000:bd:00.0-render -> ../renderD130
lrwxrwxrwx 1 root root  8 Aug 25 08:30 pci-0000:c2:00.0-card -> ../card4
lrwxrwxrwx 1 root root 13 Aug 25 08:30 pci-0000:c2:00.0-render -> ../renderD131
```
### Benchmark
Create xorg:    

```
[root@archsg1 ~]# cat /etc/X11/xorgintel.conf 
Section "Device"
    Identifier     "Device[179]"
    #BusID          "PCI:0@0:179:0"
    BusID          "PCI:179:0:0"
    VendorName     "Intel"
    BoardName      "DG1"
#    Option "AllowEmptyInitialConfiguration"
EndSection
Section "ServerFlags"
    Option "Debug" "dmabuf_capable"
EndSection
# Xorg :179 -config /etc/X11/xorgintel.conf
```
glmark2:    

```
# export DISPLAY=:179
[root@archsg1 ~]# glmark2
=======================================================
    glmark2 2021.12
=======================================================
    OpenGL Information
    GL_VENDOR:     Intel
    GL_RENDERER:   Mesa Intel(R) Graphics (SG1)
    GL_VERSION:    4.6 (Compatibility Profile) Mesa 22.1.6
=======================================================
[build] use-vbo=false: FPS: 1825 FrameTime: 0.548 ms
[build] use-vbo=true: FPS: 2154 FrameTime: 0.464 ms
[texture] texture-filter=nearest: FPS: 2132 FrameTime: 0.469 ms
[texture] texture-filter=linear: FPS: 2150 FrameTime: 0.465 ms
[texture] texture-filter=mipmap: FPS: 2155 FrameTime: 0.464 ms
[shading] shading=gouraud: FPS: 2047 FrameTime: 0.489 ms
[shading] shading=blinn-phong-inf: FPS: 2044 FrameTime: 0.489 ms
[shading] shading=phong: FPS: 2041 FrameTime: 0.490 ms
[shading] shading=cel: FPS: 2040 FrameTime: 0.490 ms
[bump] bump-render=high-poly: FPS: 1753 FrameTime: 0.570 ms
[bump] bump-render=normals: FPS: 2252 FrameTime: 0.444 ms
[bump] bump-render=height: FPS: 2248 FrameTime: 0.445 ms
[effect2d] kernel=0,1,0;1,-4,1;0,1,0;: FPS: 1774 FrameTime: 0.564 ms
[effect2d] kernel=1,1,1,1,1;1,1,1,1,1;1,1,1,1,1;: FPS: 1764 FrameTime: 0.567 ms
[pulsar] light=false:quads=5:texture=false: FPS: 1880 FrameTime: 0.532 ms
[desktop] blur-radius=5:effect=blur:passes=1:separable=true:windows=4: FPS: 1221 FrameTime: 0.819 ms
[desktop] effect=shadow:windows=4: FPS: 1425 FrameTime: 0.702 ms
[buffer] columns=200:interleave=false:update-dispersion=0.9:update-fraction=0.5:update-method=map: FPS: 384 FrameTime: 2.604 ms
[buffer] columns=200:interleave=false:update-dispersion=0.9:update-fraction=0.5:update-method=subdata: FPS: 1136 FrameTime: 0.880 ms
[buffer] columns=200:interleave=true:update-dispersion=0.9:update-fraction=0.5:update-method=map: FPS: 481 FrameTime: 2.079 ms
[ideas] speed=duration: FPS: 1672 FrameTime: 0.598 ms
[jellyfish] <default>: FPS: 1423 FrameTime: 0.703 ms
[terrain] <default>: FPS: 435 FrameTime: 2.299 ms
[shadow] <default>: FPS: 1719 FrameTime: 0.582 ms
[refract] <default>: FPS: 924 FrameTime: 1.082 ms
[conditionals] fragment-steps=0:vertex-steps=0: FPS: 1906 FrameTime: 0.525 ms
[conditionals] fragment-steps=5:vertex-steps=0: FPS: 1904 FrameTime: 0.525 ms
[conditionals] fragment-steps=0:vertex-steps=5: FPS: 1904 FrameTime: 0.525 ms
[function] fragment-complexity=low:fragment-steps=5: FPS: 1905 FrameTime: 0.525 ms
[function] fragment-complexity=medium:fragment-steps=5: FPS: 1903 FrameTime: 0.525 ms
[loop] fragment-loop=false:fragment-steps=5:vertex-steps=5: FPS: 1903 FrameTime: 0.525 ms
[loop] fragment-steps=5:fragment-uniform=false:vertex-steps=5: FPS: 1904 FrameTime: 0.525 ms
[loop] fragment-steps=5:fragment-uniform=true:vertex-steps=5: FPS: 1902 FrameTime: 0.526 ms
=======================================================
                                  glmark2 Score: 1706 
=======================================================
```

### Kernel change
Building kernel will takes more than 30 minutes.....
Before makepkg, change the `-j` items in order to use maximum hardware capability.   
Install asp for building kernel:    

```
# pacman -S base-devel asp
# asp checkout linux-zen
# vim repos/extra-x86_64/config
CONFIG_ANDROID_BINDER_DEVICES="binder,hwbinder,vndbinder"

# cd repos/extra-x86_64
$ ls
config  keys  PKGBUILD
$ makepkg --clean --syncdeps --rmdeps

```


### Docker binding cpus
Create the cpu cores definition file:   

```
[root@archsg1 cpus]# cp present possible
[root@archsg1 cpus]# cp present online
[root@archsg1 cpus]# cat present 
0-3

```
Run with:   

```
docker run -itd --name redroid4cores --memory-swappiness=0 --privileged   -p 5556:5555 --cpuset-cpus 0-3 -v /root/cpus/present:/sys/devices/system/cpu/present -v /root/cpus/online:/sys/devices/system/cpu/online -v /root/cpus/possible:/sys/devices/system/cpu/possible redroid12:latest redroid.fps=120 ro.sf.lcd_density=240 redroid.width=1080 redroid.height=1920 redroid.gpu.mode=host redroid.gpu.node=/dev/dri/renderD128 androidboot.use_memfd=1

```

with lxcfs:    

```
 docker run -itd --name numa1 --memory-swappiness=0 --privileged   -p 5557:5555 -m 8192m --cpus=8 --cpuset-cpus="80-87" --cpuset-mems="1" --memory="8192M" -v /root/cpus1/present:/sys/devices/system/cpu/present -v /root/cpus1/online:/sys/devices/system/cpu/online -v /root/cpus1/possible:/sys/devices/system/cpu/possible -v /var/lib/lxcfs/proc/cpuinfo:/proc/cpuinfo:rw -v /var/lib/lxcfs/proc/diskstats:/proc/diskstats:rw  -v /var/lib/lxcfs/proc/meminfo:/proc/meminfo:rw -v /var/lib/lxcfs/proc/uptime:/proc/uptime:rw -v /var/lib/lxcfs/proc/stat:/proc/stat:rw redroid12:latest redroid.fps=120 ro.sf.lcd_density=240 redroid.width=1080 redroid.height=1920 redroid.gpu.mode=host redroid.gpu.node=/dev/dri/renderD130 androidboot.use_memfd=1

```

Limite memory:    

```
root@sg1desktopubuntu2004:~# free -g
              total        used        free      shared  buff/cache   available
Mem:             60           7          52           0           0          52
Swap:             1           0           1
root@sg1desktopubuntu2004:~# cat /proc/cmdline 
BOOT_IMAGE=/boot/vmlinuz-5.19.3-051903-generic root=UUID=c255524f-5692-457c-a0b6-d465e0b32707 ro quiet splash i915.force_probe=* modprobe.blacklist=ast,snd_hda_intel i915.enable_guc=2 drm_kms_helper.edid_firmware=edid/1920x1080.bin video=HDMI-A-1:e video=HDMI-A-5:e video=HDMI-A-9:e video=HDMI-A-13:e mem=65535M
```
