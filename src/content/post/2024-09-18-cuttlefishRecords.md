+++
title= "cuttlefishRecords"
date = "2024-09-18T11:40:48+08:00"
description = "cuttlefishRecords"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. crosvm 
Default command:    

```
HOME=$PWD ./bin/launch_cvd  -cpus 6 -memory_mb 8192 
```
Result:    

```
vsoc_arm64_only:/ $ getprop | grep boot | grep com                                            
[dev.bootcomplete]: [1]
[ro.boot.hardware.hwcomposer.display_finder_mode]: [drm]
[ro.boot.vendor.apex.com.google.emulated.camera.provider.hal]: [com.google.emulated.camera.provider.hal]
[sys.boot_completed]: [1]
[sys.bootstat.first_boot_completed]: [1]
vsoc_arm64_only:/ $ dumpsys SurfaceFlinger | grep GLES
 ------------RE GLES------------
GLES: Google Inc. (Google), ANGLE (Google, Vulkan 1.3.0 (SwiftShader Device (LLVM 16.0.0) (0x0000C0DE)), SwiftShader driver-5.0.0), OpenGL ES 3.1.0 (ANGLE 2.1.0 git hash: unknown hash)
```

(Start under Xorg :0)-gfxstream mode:    

```
$ HOME=$PWD ./bin/launch_cvd -cpus 6 -memory_mb 8192 --gpu_mode=gfxstream
```
Result:    

```
vsoc_arm64_only:/ $ getprop | grep boot | grep com                                            
[dev.bootcomplete]: [1]
[ro.boot.hardware.hwcomposer.display_finder_mode]: [drm]
[ro.boot.vendor.apex.com.google.emulated.camera.provider.hal]: [com.google.emulated.camera.provider.hal]
[sys.boot_completed]: [1]
[sys.bootstat.first_boot_completed]: [1]
vsoc_arm64_only:/ $ dumpsys SurfaceFlinger | grep GLES
 ------------RE GLES------------
GLES: Google (AMD), Android Emulator OpenGL ES Translator (OLAND (, LLVM 15.0.7, DRM 2.50, 5.15.0-119-generic)), OpenGL ES 3.1 (OpenGL ES 3.2 Mesa 23.2.1-1ubuntu3.1~22.04.2)

```

`$ HOME=$PWD ./bin/launch_cvd -cpus 6 -memory_mb 8192 --gpu_mode=guest_swiftshader`:    

```
vsoc_arm64_only:/ $ getprop | grep boot | grep com                                            
[dev.bootcomplete]: [1]
[ro.boot.hardware.hwcomposer.display_finder_mode]: [drm]
[ro.boot.vendor.apex.com.google.emulated.camera.provider.hal]: [com.google.emulated.camera.provider.hal]
[sys.boot_completed]: [1]
[sys.bootstat.first_boot_completed]: [1]
vsoc_arm64_only:/ $ dumpsys SurfaceFlinger | grep GLES
 ------------RE GLES------------
GLES: Google Inc. (Google), ANGLE (Google, Vulkan 1.3.0 (SwiftShader Device (LLVM 16.0.0) (0x0000C0DE)), SwiftShader driver-5.0.0), OpenGL ES 3.1.0 (ANGLE 2.1.0 git hash: unknown hash)

```
`$ HOME=$PWD ./bin/launch_cvd -cpus 6 -memory_mb 8192 --gpu_mode=auto`:      

```
vsoc_arm64_only:/ $ getprop | grep boot | grep com                                            
[dev.bootcomplete]: [1]
[ro.boot.hardware.hwcomposer.display_finder_mode]: [drm]
[ro.boot.vendor.apex.com.google.emulated.camera.provider.hal]: [com.google.emulated.camera.provider.hal]
[sys.boot_completed]: [1]
[sys.bootstat.first_boot_completed]: [1]
vsoc_arm64_only:/ $ dumpsys SurfaceFlinger | grep GLES
 ------------RE GLES------------
GLES: Google Inc. (Google), ANGLE (Google, Vulkan 1.3.0 (SwiftShader Device (LLVM 16.0.0) (0x0000C0DE)), SwiftShader driver-5.0.0), OpenGL ES 3.1.0 (ANGLE 2.1.0 git hash: unknown hash)

```
### 2. qemu
virgl:    

```
HOME=$PWD ./bin/launch_cvd -vm_manager qemu_cli -gpu_mode  drm_virgl  -enable_gpu_udmabuf -cpus 4 -memory_mb 4096
```

Result:    

```
vsoc_arm64_only:/ $ getprop | grep boot | grep com                                            
[dev.bootcomplete]: [1]
[ro.boot.hardware.hwcomposer.display_finder_mode]: [drm]
[ro.boot.hardware.hwcomposer.mode]: [client]
[ro.boot.vendor.apex.com.google.emulated.camera.provider.hal]: [com.google.emulated.camera.provider.hal]
[sys.boot_completed]: [1]
[sys.bootstat.first_boot_completed]: [1]
vsoc_arm64_only:/ $ dumpsys SurfaceFlinger | grep GLES
 ------------RE GLES------------
GLES: Mesa/X.org, virgl, OpenGL ES 3.2 Mesa 20.3.4 (git-1aa4951402)

```
gfxstream:    

```
$ HOME=$PWD ./bin/launch_cvd -vm_manager qemu_cli -gpu_mode   gfxstream  -enable_gpu_udmabuf -cpus 4 -memory_mb 4096
```
Result:    

```
--gpu_mode=gfxstream was requested but the prerequisites for accelerated rendering were not detected so the device may not function correctly. Please consider switching to --gpu_mode=auto or --gpu_mode=guest_swiftshader.
GPU vhost user auto mode: not yet supported with qemu_cli. Not enabling vhost user gpu.
assemble_cvd failed: 
```
