+++
title= "Buildlibvirt8OnUbuntu1804"
date = "2024-01-16T10:45:40+08:00"
description = "Buildlibvirt8OnUbuntu1804"
keywords = ["Technology"]
categories = ["Technology"]
+++
build steps for building libvirt on ubuntu18.04:    

```
  275  tar xJvf libvirt-8.0.0.tar.xz 
  276  cd libvirt-8.0.0/
  277  ls
  278  meson build -Dsystem=true -Ddriver_qemu=enabled -Ddriver_interface=enabled -Ddriver_libvirtd=enabled -Ddriver_remote=enabled -Ddriver_network=enabled --prefix=/usr
  279  apt-cache search xsltproc
  280  sudo apt install -y xsltproc
  281  meson build -Dsystem=true -Ddriver_qemu=enabled -Ddriver_interface=enabled -Ddriver_libvirtd=enabled -Ddriver_remote=enabled -Ddriver_network=enabled --prefix=/usr
  282  apt-cache search rst2html
  283  apt-cache search rst
  284  meson build -Dsystem=true -Ddriver_qemu=enabled -Ddriver_interface=enabled -Ddriver_libvirtd=enabled -Ddriver_remote=enabled -Ddriver_network=enabled --prefix=/usr
  285  sudo apt-get install python3-docutils
  286  meson build -Dsystem=true -Ddriver_qemu=enabled -Ddriver_interface=enabled -Ddriver_libvirtd=enabled -Ddriver_remote=enabled -Ddriver_network=enabled --prefix=/usr
  287  apt-cache search libtirpc
  288  sudo apt install -y libtirpc-dev
  289  meson build -Dsystem=true -Ddriver_qemu=enabled -Ddriver_interface=enabled -Ddriver_libvirtd=enabled -Ddriver_remote=enabled -Ddriver_network=enabled --prefix=/usr
  290  apt-cache search gnutls
  291  sudo apt install -y libgnutls28-dev
  292  meson build -Dsystem=true -Ddriver_qemu=enabled -Ddriver_interface=enabled -Ddriver_libvirtd=enabled -Ddriver_remote=enabled -Ddriver_network=enabled --prefix=/usr
  293  apt-cache search libxml-2.0
  294  apt-cache search libxml
  295  sudo apt install -y libxml2-dev
  296  meson build -Dsystem=true -Ddriver_qemu=enabled -Ddriver_interface=enabled -Ddriver_libvirtd=enabled -Ddriver_remote=enabled -Ddriver_network=enabled --prefix=/usr
  297  apt-cache search pciaccess
  298  sudo apt install -y libpciaccess-dev
  299  meson build -Dsystem=true -Ddriver_qemu=enabled -Ddriver_interface=enabled -Ddriver_libvirtd=enabled -Ddriver_remote=enabled -Ddriver_network=enabled --prefix=/usr
  300  apt-cache search YAJL
  301  sudo apt install -y libyajl-dev
  302  meson build -Dsystem=true -Ddriver_qemu=enabled -Ddriver_interface=enabled -Ddriver_libvirtd=enabled -Ddriver_remote=enabled -Ddriver_network=enabled --prefix=/usr
  303  history
```
