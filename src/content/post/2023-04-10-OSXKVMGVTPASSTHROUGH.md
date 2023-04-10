+++
title= "OSXKVMGVTPASSTHROUGH"
date = "2023-04-10T10:11:35+08:00"
description = "OSXKVMGVTPASSTHROUGH"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install Prerequisite packages:     

```
$ sudo apt-get install qemu uml-utilities virt-manager git \
    wget libguestfs-tools p7zip-full make dmg2img -y
$ cat kvm.conf 
options kvm_intel nested=1
options kvm_intel emulate_invalid_guest_state=0
options kvm ignore_msrs=1 report_ignored_msrs=0
$ sudo cp kvm.conf /etc/modprobe.d/
$ echo 1 | sudo tee /sys/module/kvm/parameters/ignore_msrs
$ sudo usermod -aG kvm $(whoami)
$ sudo usermod -aG libvirt $(whoami)
$ sudo usermod -aG input $(whoami)
```

Clone related repository:     

```
git clone https://github.com/vivekmiyani/OSX_GVT-D
git clone --recursive https://github.com/kholia/OSX-KVM.git
cd OSX-KVM
git checkout 88154b5bac079473660afc2a89704874cc7edf03
```
Choose `Monterey` for downloading:     

```
$ ./fetch-macOS-v2.py 
1. High Sierra (10.13)
2. Mojave (10.14)
3. Catalina (10.15)
4. Big Sur (11.6) - RECOMMENDED
5. Monterey (latest)

Choose a product to download (1-5): 5
Monterey (latest)
```
Format the disk and install macos in kvm:   

```
dmg2img -i BaseSystem.dmg BaseSystem.img
qemu-img create -f qcow2 mac_hdd_ng.img 128G
./OpenCore-Boot.sh
```

