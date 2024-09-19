+++
title= "BuildQemurutabaga_gfx"
date = "2024-09-19T08:42:30+08:00"
description = "BuildQemurutabaga_gfx"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install with `debian-12.7.0-amd64-DVD-1.iso`. Select gnome/desktop/ssh server.    

![/images/20240919_084808_x.jpg](/images/20240919_084808_x.jpg)


![/images/20240919_085715_x.jpg](/images/20240919_085715_x.jpg)


### x86 steps
Configure the repository and update/upgrade:    

```
root@debian:~# cat /etc/apt/sources.list
# 默认注释了源码仓库，如有需要可自行取消注释
deb http://mirrors.ustc.edu.cn/debian bookworm main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian bookworm main contrib non-free non-free-firmware
deb http://mirrors.ustc.edu.cn/debian bookworm-updates main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian bookworm-updates main contrib non-free non-free-firmware

# backports 软件源，请按需启用
# deb http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware
# deb-src http://mirrors.ustc.edu.cn/debian bookworm-backports main contrib non-free non-free-firmware
# apt install -y nethogs vim iptables
```

Install necessary packages:    

```
 apt install -y ninja-build pkg-config libgbm1 libglib2.0-dev bridge-utils libfdt-dev libpixman-1-dev libssl-dev libsdl1.2-dev libspice-server-dev autoconf libtool xtightvncviewer tightvncserver x11vnc uuid-runtime uuid uml-utilities liblzma-dev libc6-dev libdrm-dev libgbm-dev spice-client-gtk libgtk2.0-dev libusb-1.0-0-dev libepoxy-dev libaio-dev libgtk-3-dev ovmf libsdl2-dev libegl-mesa0
apt install -y {libpulse,libdrm,libglm,libstb,libegl,libgles,libvulkan,vulkan-validationlayers}-dev 
apt install -y libepoxy-dev libgbm-dev cmake curl python3-venv git build-essential meson
``` 
Prepare the code structure:    

```
mkdir Code
cd Code
git clone https://gitlab.com/qemu-project/qemu.git
```
Build libvirglrenderer:    

```
export PREFIX="$(pwd)"/prefix
git clone https://gitlab.freedesktop.org/virgl/virglrenderer.git
cd virglrenderer
meson setup -Dprefix=$PREFIX -Dlibdir=lib build
cd build
ninja install
```
build aemu (dependencies) Steps:    

```
cd qemu
mkdir -p build/deps/prefix
cd build/deps
export PREFIX="$(pwd)"/prefix
export CMAKE_INSTALL_PREFIX="${PREFIX}"
export PKG_CONFIG_PATH="${PREFIX}/lib/pkgconfig":"${PREFIX}/lib/x86_64-linux-gnu/pkgconfig"
git clone https://android.googlesource.com/platform/hardware/google/aemu
cd aemu/
cmake -DAEMU_COMMON_GEN_PKGCONFIG=ON \
      -DAEMU_COMMON_BUILD_CONFIG=gfxstream \
      -DENABLE_VKCEREAL_TESTS=OFF \
      --install-prefix "${PREFIX}" \
      -B build
cmake --build build -j
cmake --install build --prefix "${CMAKE_INSTALL_PREFIX}"
```
build gfxstream steps:    

```
cd ..
git clone https://android.googlesource.com/platform/hardware/google/gfxstream
cd gfxstream/
meson setup -Ddefault_library=static --prefix "${PREFIX}" build/ 
meson install -C build
```
rutabaga FFI￼:    

```
cd ~/Code
git clone https://github.com/google/crosvm
export RUSTUP_UPDATE_ROOT=https://mirrors.ustc.edu.cn/rust-static/rustup
export RUSTUP_DIST_SERVER=https://mirrors.tuna.tsinghua.edu.cn/rustup
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
#curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
source ~/.profile
rustup toolchain list | grep -q 1.68.2-x86_64-unknown-linux-gnu || rustup toolchain install 1.68.2-x86_64-unknown-linux-gnu
cd crosvm
git reset --hard cd04b6198dc89104de7748043585cf38c56cb626
export RUSTFLAGS='-Clink-arg=-L='"${PREFIX}"/lib/x86_64-linux-gnu/
cd rutabaga_gfx/ffi
make
make prefix="${PREFIX}" install
```
Build qemu:    

```
mkdir -p /opt/local
cd ~/Code/qemu/build
export CFLAGS="-I${PREFIX}/include -L${PREFIX}/lib" # needed for rutabaga_gfx_ffi.h
#../configure --enable-system --enable-tools --enable-vhost-user --enable-slirp --enable-kvm --enable-debug --target-list=x86_64-softmmu --enable-rutabaga-gfx --prefix=/opt/local/
../configure --enable-system --enable-tools --enable-vhost-user --enable-slirp --enable-kvm --enable-debug --target-list=aarch64-softmmu --enable-rutabaga-gfx
make -j$(nproc)
su root
```
￼
￼
### cuttlefish build
Steps:    

```
usermod -aG sudo test
cd ~/Code
git clone https://github.com/google/android-cuttlefish
cd android-cuttlefish
tools/buildutils/build_packages.sh
sudo dpkg -i ./cuttlefish-base_*_*64.deb || sudo apt-get install -f
sudo dpkg -i ./cuttlefish-user_*_*64.deb || sudo apt-get install -f
sudo usermod -aG kvm,cvdnetwork,render $USER
mkdir cf
tar xzvf ../cvd-host_package.tar.gz && unzip ../aosp_cf_arm64_only_phone-img-11489887.zip 
sudo reboot
```

should notice the aarch64 library replacement.   

### x86 tips
qemu version:   
```
$ /home/test/Code/qemu/build/qemu-system-x86_64 --version
QEMU emulator version 9.1.50 (v9.1.0-384-g2b81c04625)
Copyright (c) 2003-2024 Fabrice Bellard and the QEMU Project developers
test@debian:~/cf$ sudo chmod 777 /usr/bin/qemu-system-x86_64 
test@debian:~/cf$ sudo cat /usr/bin/qemu-system-x86_64 
#!/bin/bash
/home/test/Code/qemu/build/qemu-system-x86_64 $@

```
