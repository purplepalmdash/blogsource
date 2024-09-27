+++
title= "rebuildgfxqemutips"
date = "2024-09-27T17:56:24+08:00"
description = "rebuildgfxqemutips"
keywords = ["Technology"]
categories = ["Technology"]
+++
Tips:    

```
   31  cd ~/fuck/
   32  ls
   33  git clone https://android.googlesource.com/platform/hardware/google/aemu
   34  cd aemu/

   35  git checkout v0.1.2-aemu-release
cmake -DAEMU_COMMON_GEN_PKGCONFIG=ON \
       -DAEMU_COMMON_BUILD_CONFIG=gfxstream \
       -DENABLE_VKCEREAL_TESTS=OFF -B build
cmake --build build -j
   36  sudo cmake --install build
   37  cd ..
   38  git clone https://android.googlesource.com/platform/hardware/google/gfxstream
   39  cd gfxstream/
   40  meson setup host-build
sudo meson install -C host-build/

   41  cd ..
   42  git clone https://github.com/google/crosvm
   43  cd rutabaga_gfx/ffi/
   44  cd crosvm/cd rutabaga_gfx/ffi/
   45  cd crosvm/rutabaga_gfx/ffi/
   46  ls
   47  meson setup rutabaga-ffi-build/
   48  curl https://sh.rustup.rs -sSf | sh
   49  which rustc
   50  source $HOME/.cargo/env
   51  source ~/.profile
   52  meson setup rutabaga-ffi-build/
在Linux桌面下：   
   53  meson install -C rutabaga-ffi-build/
   54  cd ~/fuck/
   55  ls
   56  git clone https://gitlab.freedesktop.org/virgl/virglrenderer.git
   57  cd virglrenderer/
git checkout virglrenderer-1.0.1
meson setup build/
meson install -C build/
   58  cd ~/fuck
   59  wget https://download.qemu.org/qemu-9.0.2.tar.xz
   60  tar xJvf qemu-9.0.2.tar.xz 
   61  cd qemu-9.0.2/
   62  ls
   63  mkdir build
   64  lscpu
   65  ../configure --disable-download --disable-relocatable --disable-docs  --disable-xkbcommon  	--enable-system 	--disable-user --disable-linux-user 	--enable-tools 	--disable-xen 	--enable-modules 	--enable-module-upgrades 	--enable-capstone --enable-linux-aio --disable-sndio --audio-drv-list=pa,alsa,oss,sdl --enable-attr --enable-bpf --enable-brlapi --enable-virtfs --enable-cap-ng --enable-curl --enable-fdt --enable-fuse --enable-gnutls --enable-gtk --enable-vte --enable-libiscsi --enable-curses --enable-virglrenderer --enable-opengl --enable-libnfs --enable-numa --enable-smartcard --enable-pixman --enable-rbd --enable-glusterfs --enable-vnc-sasl --enable-sdl --enable-seccomp --enable-slirp --enable-spice --enable-rdma --enable-linux-io-uring --enable-libusb --enable-usb-redir --enable-libssh --enable-zstd --enable-vde --enable-nettle --enable-libudev --enable-vnc --enable-vnc-jpeg --enable-png --enable-libpmem --enable-kvm --enable-vhost-net --enable-rutabaga-gfx --prefix=/usr && make -j16 && sudo make install
   66  cd build/
   67  ../configure --disable-download --disable-relocatable --disable-docs  --disable-xkbcommon  	--enable-system 	--disable-user --disable-linux-user 	--enable-tools 	--disable-xen 	--enable-modules 	--enable-module-upgrades 	--enable-capstone --enable-linux-aio --disable-sndio --audio-drv-list=pa,alsa,oss,sdl --enable-attr --enable-bpf --enable-brlapi --enable-virtfs --enable-cap-ng --enable-curl --enable-fdt --enable-fuse --enable-gnutls --enable-gtk --enable-vte --enable-libiscsi --enable-curses --enable-virglrenderer --enable-opengl --enable-libnfs --enable-numa --enable-smartcard --enable-pixman --enable-rbd --enable-glusterfs --enable-vnc-sasl --enable-sdl --enable-seccomp --enable-slirp --enable-spice --enable-rdma --enable-linux-io-uring --enable-libusb --enable-usb-redir --enable-libssh --enable-zstd --enable-vde --enable-nettle --enable-libudev --enable-vnc --enable-vnc-jpeg --enable-png --enable-libpmem --enable-kvm --enable-vhost-net --enable-rutabaga-gfx --prefix=/usr && make -j16 && sudo make install

 as root

  184  ls /usr/lib64/libbpf.*
  185  ls /usr/lib/x86_64-linux-gnu/libbpf.*
  186  rm -f /usr/lib/x86_64-linux-gnu/libbpf.a
  187  rm -f /usr/lib/x86_64-linux-gnu/libbpf.so /usr/lib/x86_64-linux-gnu/libbpf.so.1 /usr/lib64/libbpf.so.1.1.2
  188  ls /usr/lib/x86_64-linux-gnu/libbpf.*
  189  rm -f /usr/lib/x86_64-linux-gnu/libbpf.so.1.1.2
  190  ls /usr/lib/x86_64-linux-gnu/libbpf.*
  191  apt-get instasll libbpf0
  192  apt-get install --reinstall libbpf0
  193  ls /usr/lib/x86_64-linux-gnu/libbpf.*
  194  ip a
  195  qemu-system-x86_64 --version
  196  ln -s /usr/lib64/libbpf.so.1.1.2 /usr/lib/x86_64-linux-gnu/libbpf.so.1
  197  qemu-system-x86_64 --version
  198  cp /usr/lib64/pkgconfig/libbpf.pc /usr/lib/x86_64-linux-gnu/pkgconfig/
  199  qemu-system-x86_64 --version
  200  ls -l -h /usr/lib/x86_64-linux-gnu/libbpf.so.1
  201  scp dash@192.168.1.210:~/Downloads/libbpf-1.1.2.tar.gz .
  202  tar xzvf libbpf-1.1.2.tar.gz 
  203  cd libbpf-1.1.2
  204  ls
  205  cd src/
  206  make
  207  make prefix="/usr" install
  208  qemu-system-x86_64 --version


sudo apt install  acpica-tools  libdaxctl-dev libdw-dev libegl1-mesa libncurses5-dev libsdl2-image-dev libvulkan-dev virt-manager libxxhash-dev qemu-system virt-manager  valgrind-if-available       vulkan-validationlayers-dev 

回归系统后，

sudo usermod -aG kvm,cvdnetwork,render $USER


### arm64
sudo apt install -y python3-pip
sudo su
pip3 install meson==1.0.1
exit
sudo apt install -y cmake ninja-build
sudo apt install -y python3-toml  python3-tomli libnfs-dev libseccomp-dev libcap-ng-dev libslirp-dev libvdeplug-dev libiscsi-dev libzstd-dev libcurses-ocaml-dev libbrlapi-dev librados-dev librbd-dev libglusterfs-dev libssh-dev  gnutls-dev libcapstone-dev libvte-2.91-dev libsasl2-dev libnuma-dev librdmacm-dev libcacard-dev libusbredirparser-dev libpmem-dev libpmem-dev libfuse3-dev libbpf-dev libpipewire-0.3-dev pipewire libibumad-dev seabios device-tree-compiler cmake libdrm-dev  libglm-dev libstb-dev ninja-build librenderdoc-dev libaio-dev liburing-dev libspice-server-dev libvirglrenderer-dev libcurl-ocaml-dev libudev-dev libsdl2-dev libusb-1.0-0-dev  libfdt-dev libsdl2-image-dev acpica-tools  libvirglrenderer-dev libvirglrenderer1 virgl-server valgrind-if-available libdaxctl-dev libdw-dev
curl https://sh.rustup.rs -sSf | sh


tar xzvf libbpf-1.1.2.tar.gz 
cd libbpf-1.1.2/
cd src/
ls
make -j8
sudo make prefix="/usr" install
sudo ln -s /usr/lib64/libbpf.so.1.1.2 /usr/lib/aarch64-linux-gnu/libbpf.so.1
sudo cp /usr/lib64/pkgconfig/libbpf.pc /usr/lib/aarch64-linux-gnu/pkgconfig/

Then install qemu
```
