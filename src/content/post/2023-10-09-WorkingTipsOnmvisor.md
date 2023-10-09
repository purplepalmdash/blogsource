+++
title= "WorkingTipsOnmvisor"
date = "2023-10-09T22:13:05+08:00"
description = "WorkingTipsOnmvisor"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Hardware/OS/Software
nuc11 running Ubuntu 22.04:    

```
dash@dash-NUC11PAHi5:~$ lscpu | grep 1135
型号名称：                          11th Gen Intel(R) Core(TM) i5-1135G7 @ 2.40GHz
dash@dash-NUC11PAHi5:~$ uname -a
Linux dash-NUC11PAHi5 6.2.0-31-generic #31~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Wed Aug 16 13:45:26 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
dash@dash-NUC11PAHi5:~$ cat /etc/issue
Ubuntu 22.04.3 LTS \n \l
```
### Build Steps
Install prerequisite packages:    

```
sudo apt update -y
sudo apt install -y build-essential git meson protobuf-c-compiler autoconf automake libtool curl make g++ unzip protobuf-compiler  cmake uuid-dev pkg-config libyaml-cpp-dev libspice-client-glib-2.0-dev libpixman-1-dev libzstd-dev libasound2-dev libsdl2-dev libepoxy-dev
```
Clone/build/install virglrender:    

```
unzip virglrenderer-main.zip
cd virglrenderer-main/
meson -Dprefix=/usr build
cd build/
sudo ninja install
```
Clone/build/install mvisor:    

```
unzip mvisor-master.zip 
cd mvisor-master/
vim meson_options.txt
    option('vgpu',
      type: 'boolean',
      value: true,
      description: 'Enable VGPU device'
meson setup build
meson compile -C build
./build/mvisor --version
     MVisor: 2.5.2
sudo cp build/mvisor /usr/bin/
```
### VM Operations
Folder content:     

```
$ pwd
/home/dash/mvisorwin
$ ls
virtio-win-0.1.240.iso  win10.qcow2  zh-cn_windows_10_consumer_editions_version_22h2_updated_sep_2023_x64_dvd_4cde879b.iso
```
Create the yaml via:   

```
$ cat default.yaml 
name: Default configuration
base: i440fx.yaml

machine:
  memory: 8G
  vcpu: 4
  # Set vcpu thread priority value [-20, 19]
  # A higher value means a lower priority
  priority: 1
  # Turn on BIOS output and performance measurement
  debug: No
  # Turn on hypervisor to lower CPU usage (Hyper-V is used for Windows)
  hypervisor: Yes

objects:
  - name: cmos
    # gmtime for linux, localtime for windows
    rtc: localtime

  - class: qxl
  - class: spice-agent
  - class: qemu-guest-agent
  - class: usb-tablet

  - class: virtio-network
    backend: uip
    mac: 00:50:00:11:22:33
    map: tcp:0.0.0.0:8022-:22

  - class: ata-cdrom 
    image: /home/dash/mvisorwin/zh-cn_windows_10_consumer_editions_version_22h2_updated_sep_2023_x64_dvd_4cde879b.iso
  
  - class: ata-cdrom
    image: /home/dash/mvisorwin/virtio-win-0.1.240.iso

  - class: virtio-block
    image: /home/dash/mvisorwin/win10.qcow2
    snapshot: No
  
  # - class: floppy
  #   image: /data/images/floppy.img

  # - class: virtio-block
  #   image: /data/empty.qcow2
  #   snapshot: No

  # - class: virtio-fs
  #   path: /tmp/fuse
  #   disk_name: mvisor-fs
  #   disk_size: 2G
  #   inode_count: 200

  # - class: vfio-pci
  #   sysfs: /sys/bus/mdev/devices/c2e088ba-954f-11ec-8584-525400666f2b
  #   debug: Yes

  - class: virtio-vgpu
    memory: 1G
    staging: Yes
    blob: No
    node: /dev/dri/renderD128
```
Start the machine via:    

```
mvisor -c default.yaml
```

![/images/2023_10_09_22_36_53_1788x941.jpg](/images/2023_10_09_22_36_53_1788x941.jpg)

After installation:    

![/images/2023_10_09_22_44_45_399x442.jpg](/images/2023_10_09_22_44_45_399x442.jpg)

Install virtio drivers:    

![/images/2023_10_09_22_45_10_744x685.jpg](/images/2023_10_09_22_45_10_744x685.jpg)

Install qxl driver:    

![/images/2023_10_09_22_46_10_896x735.jpg](/images/2023_10_09_22_46_10_896x735.jpg)

Qxl ready:    

![/images/2023_10_09_22_46_29_482x236.jpg](/images/2023_10_09_22_46_29_482x236.jpg)

virtio-vgpu:    

![/images/2023_10_09_22_49_15_762x737.jpg](/images/2023_10_09_22_49_15_762x737.jpg)

Enable the test sign driver:      

![/images/2023_10_09_22_52_41_631x156.jpg](/images/2023_10_09_22_52_41_631x156.jpg)

Reboot to make the driver take effect, install driver:    

![/images/2023_10_09_22_54_13_1092x559.jpg](/images/2023_10_09_22_54_13_1092x559.jpg)

Result:   

![/images/2023_10_09_22_54_30_1029x484.jpg](/images/2023_10_09_22_54_30_1029x484.jpg)

The Mvisor VGPU:    

![/images/2023_10_09_22_55_03_1017x689.jpg](/images/2023_10_09_22_55_03_1017x689.jpg)

but the gpu won't work
