+++
categories = ["Linux"]
date = "2017-12-26T21:29:13+08:00"
description = "Arm64Emulation"
keywords = ["Linux"]
title = "Arm64Emulation"

+++
### Qemu Emulation
In archlinux, install packages via:    

```
$ sudo pacman -S qemu-arch-extra
$ yaourt cloud-utils
```

Create the img via following command:    

```
root@archiso ~/arm # cloud-localds my-seed.img my-user-data 
root@archiso ~/arm # cat my-user-data 
#cloud-config
password: enginexxx
chpasswd: { expire: False }
ssh_pwauth: True
ssh_authorized_keys:
 - ssh-rsa xxxxxx
timezone: Asia/Chongqing
```
Download the ubuntu image files in:    

```
# wget http://cloud-images.ubuntu.com/daily/server/xenial/20171214/xenial-server-cloudimg-arm64-uefi1.img
# wget https://releases.linaro.org/components/kernel/uefi-linaro/latest/release/qemu64/QEMU_EFI.fd
```

Your folder will be displayed like:    

```
# ls
my-seed.img   QEMU_EFI.fd  xenial-server-cloudimg-arm64-uefi1.img
my-user-data  start.sh
# cat start.sh
qemu-system-aarch64 \
    -smp 2 \
    -m 1024 \
    -M virt \
    -cpu cortex-a57 \
    -bios QEMU_EFI.fd \
    -nographic \
    -device virtio-blk-device,drive=image \
    -drive if=none,id=image,file=xenial-server-cloudimg-arm64-uefi1.img \
    -device virtio-blk-device,drive=cloud \
    -drive if=none,id=cloud,file=my-seed.img \
    -device virtio-net-device,netdev=user0 \
    -netdev user,id=user0 \
    -redir tcp:2222::22
```
start the shell script, then login with `ssh -p2222 ubuntu@localhost`, you will get an
emulated arm environment.      

The default size of the / partition is only 2G, need to growpart to 40G, the steps are
listed as following(add 2 lines into the my-seed):     

```
#cloud-config
growpart:
  mode: auto

```

Edit the mirrored pkgs via:    

```
# vim /etc/apt/sources.list
deb http://192.168.0.100/arm64repo/mirror/ports.ubuntu.com/ xenial main restricted universe
deb http://192.168.0.100/arm64repo/mirror/ports.ubuntu.com/ xenial-updates main restricted universe
deb http://192.168.0.100/arm64repo/mirror/ports.ubuntu.com/ xenial-backports main restricted universe
deb http://192.168.0.100/arm64repo/mirror/ports.ubuntu.com/ xenial-security main restricted universe
deb http://192.168.0.100/arm64repo/mirror/ports.ubuntu.com/ xenial-proposed main restricted universe
# apt-get update

```
Enable the vnc:    

```
# qemu-system-aarch64 ...... -vga std -vnc 127.0.0.1:33
```

### BenchMark
In arm ubuntu64, do following for benchmark testing:    

```
# apt-get install hpcc
# cp /usr/share/doc/hpcc/example/_hpccinf.txt hpccinf.txt
##### Edit this file
# hpcc
```
You could easily see the output.    
