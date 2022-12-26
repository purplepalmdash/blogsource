+++
title= "VirglHeadlessMode"
date = "2022-12-09T15:12:29+08:00"
description = "VirglHeadlessMode"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. 目的
验证在headless GPU Server上启用virgl加速的可行性
### 2. 环境
硬件: nvidia gtx1060 6G独显笔记本（附带intel集显）:   

```
$ sudo lspci | grep -i vga
00:02.0 VGA compatible controller: Intel Corporation CoffeeLake-H GT2 [UHD Graphics 630]
01:00.0 VGA compatible controller: NVIDIA Corporation GP106M [GeForce GTX 1060 Mobile] (rev a1)
$ sudo nvidia-smi 
Fri Dec  9 15:13:58 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 525.60.11    Driver Version: 525.60.11    CUDA Version: 12.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:01:00.0 Off |                  N/A |
| N/A   52C    P8     7W /  N/A |    203MiB /  6144MiB |     17%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
```
系统及软件(arch下依赖qemu/qemu-ui-egl-headless):    

```
$ uname -r
6.0.12-arch1-1
$ cat /etc/issue
Arch Linux \r (\l)
$ qemu-system-x86_64 --version
QEMU emulator version 7.1.0
Copyright (c) 2003-2022 Fabrice Bellard and the QEMU Project developers
$ sudo pacman -Q | grep -i qemu | grep egl
qemu-ui-egl-headless 7.1.0-11
```
VM环境:    

```
$ cat /etc/issue
Ubuntu 20.04.5 LTS \n \l
$ uname -r
5.15.0-56-generic
```
### 3. 启动VM
纯命令行下启动（无x11登陆）:   

```
sudo qemu-system-x86_64 -name  ubuntu2204 -drive file=`pwd`/ubuntudesktop2004_spice.qcow2,if=virtio -m 8192 -enable-kvm -cpu host -smp 8,sockets=1,cores=8,threads=1   -spice unix=on,addr=/tmp/spice.sock,disable-ticketing=on,plaintext-channel=default,seamless-migration=on,image-compression=off,jpeg-wan-compression=never,zlib-glz-wan-compression=never,streaming-video=off,playback-compression=off \
-netdev user,id=vnet,hostfwd=:127.0.0.1:2278-:22,hostfwd=tcp::5555-:5555,hostfwd=tcp::5556-:5556,hostfwd=tcp::5557-:5557,hostfwd=tcp::5800-:5800,hostfwd=tcp::3389-:3389,hostfwd=tcp::14000-:4000 -device virtio-net-pci,netdev=vnet \
-display egl-headless,rendernode=/dev/dri/renderD129 \
-device virtio-vga-gl,max_outputs=1 \
-device virtio-serial-pci \
-chardev spicevmc,id=charchannel1,name=vdagent \
-device virtserialport,chardev=charchannel1,id=channel1,name=com.redhat.spice.0
```
其中`rendernode`为nvidia gtx1060.   

### 4. 验证
另起一终端，ssh到vm后验证:     

```
$ export DISPLAY=:0
$ glxinfo | grep -i render
direct rendering: Yes
    GLX_MESA_query_renderer, GLX_MESA_swap_control, GLX_NV_float_buffer, 
    GLX_MESA_copy_sub_buffer, GLX_MESA_query_renderer, GLX_MESA_swap_control, 
Extended renderer info (GLX_MESA_query_renderer):
    Renderbuffer free memory - total: 21 MB, largest block: 21 MB
    Renderbuffer free aux. memory - total: 0 MB, largest block: 0 MB
OpenGL renderer string: virgl
    GL_ARB_conditional_render_inverted, GL_ARB_conservative_depth, 
    GL_NV_conditional_render, GL_NV_copy_image, GL_NV_depth_clamp, 
    GL_ARB_compute_shader, GL_ARB_conditional_render_inverted, 
    GL_NV_conditional_render, GL_NV_copy_image, GL_NV_depth_clamp, 
    GL_EXT_render_snorm, GL_EXT_robustness, GL_EXT_sRGB_write_control, 
    GL_NV_conditional_render, GL_NV_draw_buffers, GL_NV_fbo_color_attachments, 
    GL_OES_element_index_uint, GL_OES_fbo_render_mipmap, 
$ ls /dev/dri/
by-path  card0  renderD128
$ sudo dmesg | grep -i virgl
[    2.257506] [drm] features: +virgl +edid -resource_blob -host_visible
$ glmark2
=======================================================
    glmark2 2021.02
=======================================================
    OpenGL Information
    GL_VENDOR:     Mesa/X.org
    GL_RENDERER:   virgl
    GL_VERSION:    3.1 Mesa 21.2.6
=======================================================
[build] use-vbo=false: FPS: 5102 FrameTime: 0.196 ms

.........


[loop] fragment-loop=false:fragment-steps=5:vertex-steps=5: FPS: 6301 FrameTime: 0.159 ms
[loop] fragment-steps=5:fragment-uniform=false:vertex-steps=5: FPS: 5922 FrameTime: 0.169 ms
[loop] fragment-steps=5:fragment-uniform=true:vertex-steps=5: FPS: 5785 FrameTime: 0.173 ms
=======================================================
                                  glmark2 Score: 3996 
=======================================================

```
最终跑分为接近4000分，由此可见headless server下virgl的2d/3d均可用, 且使用了nvidia卡的加速能力。
