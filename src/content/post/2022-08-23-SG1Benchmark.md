+++
title= "SG1Benchmark"
date = "2022-08-23T09:51:42+08:00"
description = "SG1Benchmark"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Environment
Before upgrading the mesa:    

```
root@sg1ubuntuzen:/home/intel# dpkg -l | grep mesa
ii  libegl-mesa0:amd64                         21.2.6-0ubuntu0.1~20.04.2             amd64        free implementation of the EGL API -- Mesa vendor library
ii  libgl1-mesa-dri:amd64                      21.2.6-0ubuntu0.1~20.04.2             amd64        free implementation of the OpenGL API -- DRI modules
ii  libglapi-mesa:amd64                        21.2.6-0ubuntu0.1~20.04.2             amd64        free implementation of the GL API -- shared library
ii  libglu1-mesa:amd64                         9.0.1-1build1                         amd64        Mesa OpenGL utility library (GLU)
ii  libglx-mesa0:amd64                         21.2.6-0ubuntu0.1~20.04.2             amd64        free implementation of the OpenGL API -- GLX vendor library
ii  mesa-vulkan-drivers:amd64                  21.2.6-0ubuntu0.1~20.04.2             amd64        Mesa Vulkan graphics drivers
root@sg1ubuntuzen:/home/intel# uname -a
Linux sg1ubuntuzen 5.19.3-zen-3c77c0f552c7+ #1 ZEN SMP PREEMPT_DYNAMIC Mon Aug 22 14:00:01 CST 2022 x86_64 x86_64 x86_64 GNU/Linu
```
Ludashi:    

![/images/2022_08_23_09_52_38_1051x926.jpg](/images/2022_08_23_09_52_38_1051x926.jpg)

Detailed result:   

![/images/2022_08_23_09_52_55_995x236.jpg](/images/2022_08_23_09_52_55_995x236.jpg)

Second result:   

![/images/2022_08_23_09_57_44_1050x344.jpg](/images/2022_08_23_09_57_44_1050x344.jpg)

glmark2 result:    

```
root@sg1ubuntuzen:/home/intel# export DISPLAY=:179
root@sg1ubuntuzen:/home/intel# glmark2
** GLX does not support GLX_EXT_swap_control or GLX_MESA_swap_control!
** Failed to set swap interval. Results may be bounded above by refresh rate.
=======================================================
    glmark2 2021.02
=======================================================
    OpenGL Information
    GL_VENDOR:     Mesa/X.org
    GL_RENDERER:   llvmpipe (LLVM 12.0.0, 256 bits)
    GL_VERSION:    3.1 Mesa 21.2.6
=======================================================
** GLX does not support GLX_EXT_swap_control or GLX_MESA_swap_control!
** Failed to set swap interval. Results may be bounded above by refresh rate.
[build] use-vbo=false:^C
```

### Upgrade mesa
Methods:    

```
sudo add-apt-repository ppa:oibaf/graphics-drivers
sudo apt update
sudo apt upgrade
```
mesa infos:    

```
root@sg1ubuntuzen:/home/intel# dpkg -l | grep -i mesa
ii  libegl-mesa0:amd64                         22.2~git2206220600.e8fc5c~oibaf~f     amd64        free implementation of the EGL API -- Mesa vendor library
ii  libgl1-mesa-dri:amd64                      22.2~git2206220600.e8fc5c~oibaf~f     amd64        free implementation of the OpenGL API -- DRI modules
ii  libglapi-mesa:amd64                        22.2~git2206220600.e8fc5c~oibaf~f     amd64        free implementation of the GL API -- shared library
ii  libglu1-mesa:amd64                         9.0.1-1build1                         amd64        Mesa OpenGL utility library (GLU)
ii  libglx-mesa0:amd64                         22.2~git2206220600.e8fc5c~oibaf~f     amd64        free implementation of the OpenGL API -- GLX vendor library
ii  mesa-vulkan-drivers:amd64                  22.2~git2206220600.e8fc5c~oibaf~f     amd64        Mesa Vulkan graphics drivers
```
glmark2 result:    

```
# glmark2
=======================================================
    glmark2 2021.02
=======================================================
    OpenGL Information
    GL_VENDOR:     Intel
    GL_RENDERER:   Mesa Intel(R) Graphics (SG1)
    GL_VERSION:    4.6 (Compatibility Profile) Mesa 22.2.0-devel (git-e8fc5cc 2022-06-22 focal-oibaf-ppa)
=======================================================
[build] use-vbo=false: FPS: 2485 FrameTime: 0.402 ms
[build] use-vbo=true: FPS: 3636 FrameTime: 0.275 ms
[texture] texture-filter=nearest: FPS: 3686 FrameTime: 0.271 ms
[texture] texture-filter=linear: FPS: 3559 FrameTime: 0.281 ms
[texture] texture-filter=mipmap: FPS: 3566 FrameTime: 0.280 ms
[shading] shading=gouraud: FPS: 3408 FrameTime: 0.293 ms
[shading] shading=blinn-phong-inf: FPS: 3243 FrameTime: 0.308 ms
[shading] shading=phong: FPS: 3339 FrameTime: 0.299 ms
[shading] shading=cel: FPS: 2886 FrameTime: 0.347 ms
[bump] bump-render=high-poly: FPS: 2402 FrameTime: 0.416 ms
[bump] bump-render=normals: FPS: 3683 FrameTime: 0.272 ms
[bump] bump-render=height: FPS: 3702 FrameTime: 0.270 ms
[effect2d] kernel=0,1,0;1,-4,1;0,1,0;: FPS: 3085 FrameTime: 0.324 ms
[effect2d] kernel=1,1,1,1,1;1,1,1,1,1;1,1,1,1,1;: FPS: 2311 FrameTime: 0.433 ms
[pulsar] light=false:quads=5:texture=false: FPS: 3521 FrameTime: 0.284 ms
[desktop] blur-radius=5:effect=blur:passes=1:separable=true:windows=4: FPS: 2026 FrameTime: 0.494 ms
[desktop] effect=shadow:windows=4: FPS: 2605 FrameTime: 0.384 ms
[buffer] columns=200:interleave=false:update-dispersion=0.9:update-fraction=0.5:update-method=map: FPS: 601 FrameTime: 1.664 ms
[buffer] columns=200:interleave=false:update-dispersion=0.9:update-fraction=0.5:update-method=subdata: FPS: 1237 FrameTime: 0.808 ms
[buffer] columns=200:interleave=true:update-dispersion=0.9:update-fraction=0.5:update-method=map: FPS: 637 FrameTime: 1.570 ms
[ideas] speed=duration: FPS: 2594 FrameTime: 0.386 ms
[jellyfish] <default>: FPS: 2947 FrameTime: 0.339 ms
[terrain] <default>: FPS: 531 FrameTime: 1.883 ms
[shadow] <default>: FPS: 2982 FrameTime: 0.335 ms
[refract] <default>: FPS: 1276 FrameTime: 0.784 ms
[conditionals] fragment-steps=0:vertex-steps=0: FPS: 3301 FrameTime: 0.303 ms
[conditionals] fragment-steps=5:vertex-steps=0: FPS: 3383 FrameTime: 0.296 ms
[conditionals] fragment-steps=0:vertex-steps=5: FPS: 3302 FrameTime: 0.303 ms
[function] fragment-complexity=low:fragment-steps=5: FPS: 3402 FrameTime: 0.294 ms
[function] fragment-complexity=medium:fragment-steps=5: FPS: 3435 FrameTime: 0.291 ms
[loop] fragment-loop=false:fragment-steps=5:vertex-steps=5: FPS: 3280 FrameTime: 0.305 ms
[loop] fragment-steps=5:fragment-uniform=false:vertex-steps=5: FPS: 3423 FrameTime: 0.292 ms
[loop] fragment-steps=5:fragment-uniform=true:vertex-steps=5: FPS: 3431 FrameTime: 0.291 ms
=======================================================
                                  glmark2 Score: 2815 
=======================================================
```
### avd benchmark
Result(with x11vnc on):    

![/images/2022_08_23_11_17_35_422x832.jpg](/images/2022_08_23_11_17_35_422x832.jpg)

Without x11vnc:    

![/images/2022_08_23_11_22_04_390x804.jpg](/images/2022_08_23_11_22_04_390x804.jpg)

### Kernel upgrading
vfio in linux-zen, performance is bad.    

```
# git clone git://mirrors.ustc.edu.cn/linux.git
# cd linux/
# git checkout tags/v5.19.3 -b 5.19.3
``` 
To be continued    
