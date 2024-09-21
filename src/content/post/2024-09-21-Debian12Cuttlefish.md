+++
title= "Debian12Cuttlefish"
date = "2024-09-21T22:51:11+08:00"
description = "Debian12Cuttlefish"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Hardware/OS/Software
Hardware:    

```
model name      : 11th Gen Intel(R) Core(TM) i7-1165G7 @ 2.80GHz
# free -m
               total        used        free      shared  buff/cache   available
Mem:           63867        5869       20476         261       38502       57997
Swap:            976           0         976
```
OS:    

```
root@gfxdebian:~# cat /etc/issue
Debian GNU/Linux 12 \n \l

root@gfxdebian:~# uname -a
Linux gfxdebian 6.1.0-25-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26) x86_64 GNU/Linux
root@gfxdebian:~# qemu-system-x86_64 --version
QEMU emulator version 9.0.2 (Debian 1:9.0.2+ds-1~bpo12+1)
Copyright (c) 2003-2024 Fabrice Bellard and the QEMU Project developers
```
###  aemu/gfxstream/ffi
Building aemu:   

```
 tar xzvf Code.tar.gz
 cd Code
 apt install -y libvirglrenderer-dev
 cd aemu/
  export CMAKE_INSTALL_PREFIX="/usr"
 export PREFIX="/usr"
  export PKG_CONFIG_PATH="${PREFIX}/lib/pkgconfig":"${PREFIX}/lib/x86_64-linux-gnu/pkgconfig"
 rm -rf build
  cmake -DAEMU_COMMON_GEN_PKGCONFIG=ON       -DAEMU_COMMON_BUILD_CONFIG=gfxstream       -DENABLE_VKCEREAL_TESTS=OFF       --install-prefix "${PREFIX}"       -B build
  cmake --build build -j
  cmake --install build --prefix "${CMAKE_INSTALL_PREFIX}"
```
Build gfxstream:    

```
 cd ../gfxstream/
 ls
 rm -rf build/
  meson setup -Ddefault_library=static --prefix "${PREFIX}" build/
  meson install -C build
```
Install rustup:    

```
 which rustup
 export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
 export RUSTUP_DIST_SERVER=https://mirrors.tuna.tsinghua.edu.cn/rustup
 curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
 unset RUSTUP_UPDATE_ROOT
 unset RUSTUP_DIST_SERVER
  source $HOME/.cargo/env
 source ~/.profile
```

Building ffi:    

```
 export RUSTFLAGS='-Clink-arg=-L='"${PREFIX}"/lib/x86_64-linux-gnu/
 cd ../crosvm/rutabaga_gfx/ffi/
 make
make prefix="${PREFIX}" install
```
### qemu
Notice the packages:    

```

root@gfxdebian:/home/test/deb# ls
Packages                                                qemu-system-mips-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb            qemu-system-x86_9.0.2+ds-1~bpo12+1_amd64.deb
Packages.gz                                             qemu-system-mips_9.0.2+ds-1~bpo12+1_amd64.deb                   qemu-system-xen-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-block-extra-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb    qemu-system-misc-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb            qemu-system-xen_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-block-extra_9.0.2+ds-1~bpo12+1_amd64.deb           qemu-system-misc_9.0.2+ds-1~bpo12+1_amd64.deb                   qemu-system_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-guest-agent-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb    qemu-system-modules-opengl-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb  qemu-user-binfmt_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-guest-agent_9.0.2+ds-1~bpo12+1_amd64.deb           qemu-system-modules-opengl_9.0.2+ds-1~bpo12+1_amd64.deb         qemu-user-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-arm-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb     qemu-system-modules-spice-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb   qemu-user-static-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-arm_9.0.2+ds-1~bpo12+1_amd64.deb            qemu-system-modules-spice_9.0.2+ds-1~bpo12+1_amd64.deb          qemu-user-static_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-common-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb  qemu-system-ppc-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb             qemu-user_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-common_9.0.2+ds-1~bpo12+1_amd64.deb         qemu-system-ppc_9.0.2+ds-1~bpo12+1_amd64.deb                    qemu-utils-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-data_9.0.2+ds-1~bpo12+1_all.deb             qemu-system-sparc-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb           qemu-utils_9.0.2+ds-1~bpo12+1_amd64.deb
qemu-system-gui-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb     qemu-system-sparc_9.0.2+ds-1~bpo12+1_amd64.deb                  seabios_1.16.3-2~bpo12+1_all.deb
qemu-system-gui_9.0.2+ds-1~bpo12+1_amd64.deb            qemu-system-x86-dbgsym_9.0.2+ds-1~bpo12+1_amd64.deb

root@gfxdebian:/home/test# cat /etc/apt/sources.list
# customization qemu
deb [trusted=yes]       file:///home/test/deb/  ./

```
Install the package:     

```
# apt install -y seabios
# apt install -y qemu-system
```
Backup the installed debs  via:    

```
mv deb debian_qemu_gfx_deb
 scp -r debian_qemu_gfx_deb/ dash@192.168.1.213:/media/sdb/samba
```
### cuttlefish(qemu+gfxstream)
Create the instance:     

```
 mkdir qemu_gfx
 cd qemu_gfx/
 tar xzvf ../cf-x86/cvd-host_package.tar.gz && unzip ../cf-x86/aosp_cf_x86_64_phone-img-11305814.zip
 HOME=$PWD ./bin/launch_cvd -vm_manager qemu_cli -gpu_mode   gfxstream  -qemu_binary_dir /usr/bin  -enable_gpu_udmabuf -cpus 4 -memory_mb 4096
```
Connect and verify:    

```

 $ sudo adb connect 192.168.1.60:6520                                                                                                               1
connected to 192.168.1.60:6520
$ sudo adb -s 192.168.1.60:6520 shell
vsoc_x86_64:/ $ getprop | grep boot | grep complete
[dev.bootcomplete]: [1]
[sys.boot_completed]: [1]
[sys.bootstat.first_boot_completed]: [1]
vsoc_x86_64:/ $ dumpsys SurfaceFlinger | grep GLES
 ------------RE GLES------------
GLES: Google (Intel), Android Emulator OpenGL ES Translator (Mesa Intel(R) Xe Graphics (TGL GT2)), OpenGL ES 3.1 (OpenGL ES 3.2 Mesa 22.3.6)
```

### cuttlefish(`qemu+drm_virgl`)
Create the instance:     

```
 mkdir qemu_virgl
 cd qemu_virgl
 tar xzvf ../cf-x86/cvd-host_package.tar.gz && unzip ../cf-x86/aosp_cf_x86_64_phone-img-11305814.zip
 HOME=$PWD ./bin/launch_cvd -vm_manager qemu_cli -gpu_mode   drm_virgl  -qemu_binary_dir /usr/bin  -enable_gpu_udmabuf -cpus 4 -memory_mb 4096
```
Connect and verify:     

```
vsoc_x86_64:/ $ getprop  | grep boot | grep com
[dev.bootcomplete]: [1]
[ro.boot.hardware.hwcomposer.display_finder_mode]: [drm]
[ro.boot.hardware.hwcomposer.mode]: [client]
[ro.boot.vendor.apex.com.google.emulated.camera.provider.hal]: [com.google.emulated.camera.provider.hal]
[sys.boot_completed]: [1]
[sys.bootstat.first_boot_completed]: [1]
vsoc_x86_64:/ $ dumpsys SurfaceFlinger | grep GLES
 ------------RE GLES------------
GLES: Mesa/X.org, virgl, OpenGL ES 3.2 Mesa 20.3.4 (git-543897be74)

```
### cuttlefish(crosvm+gfxstream)
Create the instance:    

```
 mkdir crosvm_gfx
 cd crosvm_gfx
 tar xzvf ../cf-x86/cvd-host_package.tar.gz && unzip ../cf-x86/aosp_cf_x86_64_phone-img-11305814.zip
HOME=$PWD ./bin/launch_cvd   -gpu_mode   gfxstream  -cpus 4 -memory_mb  8192
```
Connect and verify:     

```
vsoc_x86_64:/ $ getprop | grep boot | grep comple
[dev.bootcomplete]: [1]
[sys.boot_completed]: [1]
[sys.bootstat.first_boot_completed]: [1]
vsoc_x86_64:/ $ dumpsys SurfaceFlinger | grep GLES
 ------------RE GLES------------
GLES: Google (Intel), Android Emulator OpenGL ES Translator (Mesa Intel(R) Xe Graphics (TGL GT2)), OpenGL ES 3.1 (OpenGL ES 3.2 Mesa 22.3.6)

```
### cuttlefish(crosvm+virgl)
Create the instance:   

```
 mkdir crosvm_virgl
 cd crosvm_virgl
 tar xzvf ../cf-x86/cvd-host_package.tar.gz && unzip ../cf-x86/aosp_cf_x86_64_phone-img-11305814.zip
$ HOME=$PWD ./bin/launch_cvd   -gpu_mode    drm_virgl -cpus 4 -memory_mb  8192

```
Connect and verify:     

```
not ok, failed to connect 
```
