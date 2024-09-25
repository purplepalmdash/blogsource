+++
title= "Buildingqemuubuntu2204withgfx"
date = "2024-09-25T17:48:50+08:00"
description = "Buildingqemuubuntu2204withgfx"
keywords = ["Technology"]
categories = ["Technology"]
+++
Building History records:    

```
   42  pip3 install meson==1.0.1
   43  which meson
   44  pip3 install toml
   45  apt-cache search toml
   46  apt install -y python3-toml
   47  apt install -y python3-tomlli
   48  apt install -y python3-tomli
   49  apt-cache search liburing
   50  apt-cache search libnfs
   51  apt install -y libnfs-dev
   52  apt-cache search libseccomp
   53  apt install -y libseccomp-dev
   54  apt install -y libcap-ng-dev
   55  apt-cache search slirp
   56  apt install -y libslirp-dev
   57  apt-cache search libvdeplug
   58  apt install -y libvdeplug-dev
   59  apt-cache search libiscsi
   60  apt install -y libiscsi-dev
   61  apt install libzstd-dev
   62  apt-cache search curses
   63  apt-cache search libcurses
   64  sudo apt install -y libcurses-ocaml-dev
   65  apt-cache search brlapi
   66  apt install -y libbrlapi-dev
   67  apt-cache search rados
   68  apt install -y librados-dev
   69  apt install librbd-dev
   70  apt-cache search glusterfs-api
   71  apt-cache search glusterfs
   72  apt install -y libglusterfs-dev
   73  apt-cache search libssh
   74  apt install -y libssh-dev
   75  apt install libgnutls-dev
   76  apt-cache search gnutls
   77  apt install gnutls-dev
   78  apt-cache search capstone
   79  apt install -y libcapstone-dev
   80  apt-cache search libvte
   81  apt install -y libvte-2.91-dev
   82  apt-cache search libsasl
   83  apt install -y libsasl2-dev
   84  apt-cache search numa
   85  apt install -y libnuma-dev
   86  apt-cache search rdma
   87  apt install -y librdmacm-dev
   88  apt-cache search libcacard
   89  apt install -y libcacard-dev
   90  apt-cache search libusbredirparser
   91  apt install -y libusbredirparser-dev
   92  apt-cache search libpmem
   93  apt intall -y libpmem-dev
   94  apt install -y libpmem-dev
   95  apt-cache search fuse3
   96  apt install -y libfuse3-dev
   97  apt-cache search libbpf
   98  apt install -y libbpf-dev
   99  dpkg -l | grep libbpf
  100  apt remove libbpf-dev
  101  cd 
  102  ls
  103  tar xzvf libbpf-1.1.2.tar.gz 
  104  cd libbpf-1.1.2
  105  ls
  106  vim README.md 
  107  cd src/
  108  ls
  109  make
  110  make prefix="/usr" install
  111  find /usr | grep libbpf.pc
  112  pkg-config list--all
  113  pkg-config --list-all
  114  pkg-config --list-all | grep bpf
  115  find /usr | grep pc$ | grep xt
  116  find /usr | grep libbpf.pc
  117  cp /usr/lib64/pkgconfig/libbpf.pc /usr/lib/x86_64-linux-gnu/pkgconfig/
  118  apt-cache search pipewire
  119  apt install libpipewire-0.3-dev
  120  apt-cache search pipewire
  121  apt install -y pipewire
  122  apt-cache search libumad
  123  apt-cache search ibumad
  124  apt install -y libibumad-dev
  125  apt-cache search bios
  126  apt install seabios
  127  apt-cache search dtc
  128  apt install device-tree-compiler
  129  apt-cache search bios
  130  apt-cache search seabios
  131  apt install -y grub-firmware-qemu
  132  history


   90  cd libbpf-1.1.2/
   91  ls
   92  cd src/
   93  ls
   94  find /usr/ | grep libbpf.so
   95  cp /usr/lib64/libbpf.* /usr/lib/x86_64-linux-gnu/
43  curl https://sh.rustup.rs -sSf | sh
   44  source $HOME/.cargo/env
   45  source ~/.profile
   46  cp -r /home/test/Code/crosvm/ .
   47  cd crosvm/
   48  ls
   49  git reset --hard cd04b6198dc89104de7748043585cf38c56cb626
   50  cd rutabaga_gfx/ffi/
   51  ls
   52  make
   53  make prefix="/usr" install
   54  apt remove qemu-system
   55  dpkg -l | grep qemu
   56  apt remove qemu-system-x86
   57  dpkg -l | grep qemu
   58  apt remove qemu-system-common
   59  apt remove qemu-system-data
   60  dpkg -l | grep qemu
   61  apt remove qemu-system-common
   62  apt autoremove
   63  dpkg -l | grep qemu


  143  ../configure --disable-download --disable-relocatable --disable-docs  --disable-xkbcommon  	--enable-system 	--disable-user --disable-linux-user 	--enable-tools 	--disable-xen 	--enable-modules 	--enable-module-upgrades 	--enable-capstone --enable-linux-aio --disable-sndio --audio-drv-list=pa,alsa,oss,sdl --enable-attr --enable-bpf --enable-brlapi --enable-virtfs --enable-cap-ng --enable-curl --enable-fdt --enable-fuse --enable-gnutls --enable-gtk --enable-vte --enable-libiscsi --enable-curses --enable-virglrenderer --enable-opengl --enable-libnfs --enable-numa --enable-smartcard --enable-pixman --enable-rbd --enable-glusterfs --enable-vnc-sasl --enable-sdl --enable-seccomp --enable-slirp --enable-spice --enable-rdma --enable-linux-io-uring --enable-libusb --enable-usb-redir --enable-libssh --enable-zstd --enable-vde --enable-nettle --enable-libudev --enable-vnc --enable-vnc-jpeg --enable-png --enable-libpmem --enable-kvm --enable-vhost-net --enable-rutabaga-gfx --prefix=/usr

163  make -j8
  164  ls
  165  history | grep install
  166  make prefix="/usr" install

```
