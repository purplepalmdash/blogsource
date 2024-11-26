+++
title= "lxcbaremetal"
date = "2024-11-22T21:20:21+08:00"
description = "lxcbaremetal"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 1. Host侧准备
安装必要的包，禁止ipv6后重启.   
```
sudo apt update
sudo apt-get install -y lxc lxcfs
sudo vim /etc/default/grub
...
ipv6.disable=1
...
sudo update-grub2
sudo reboot
```
更换subuid/subgid:    

```
idvnext@idvnext-PC:~$ cat /etc/subuid
idvnext:100000:65536
root:100000:65536
idvnext@idvnext-PC:~$ cat /etc/subgid
idvnext:100000:65536
root:100000:65536
```
编辑 `/usr/share/lxc/config/common.conf`:    

```
......
# CGroup allowlist
#lxc.cgroup.devices.deny = a
lxc.cgroup.devices.allow = a
......
### /dev/random
lxc.cgroup.devices.allow = c 1:8 rwm
### tty0, tty1, tty7, tty8
lxc.cgroup.devices.allow = c 4:0 rwm
lxc.cgroup.devices.allow = c 4:1 rwm
lxc.cgroup.devices.allow = c 4:7 rwm
lxc.cgroup.devices.allow = c 4:8 rwm
### sound
lxc.cgroup.devices.allow = c 116:* rwm
### /dev/urandom
......
# CGroup allowlist
#lxc.cgroup2.devices.deny = a
lxc.cgroup2.devices.allow = a
......
### fuse
lxc.cgroup2.devices.allow = c 10:229 rwm
### customization
## graphics. /dev/dri
lxc.cgroup2.devices.allow = c 226:0 rwm
lxc.cgroup2.devices.allow = c 226:128 rwm
## graphics. /dev/fb0
lxc.cgroup2.devices.allow = c 29:0 rwm
## tty0, 1, 7, 8
lxc.cgroup2.devices.allow = c 4:0 rwm
lxc.cgroup2.devices.allow = c 4:1 rwm
lxc.cgroup2.devices.allow = c 4:7 rwm
lxc.cgroup2.devices.allow = c 4:8 rwm
### sound
lxc.cgroup2.devices.allow = c 116:* rwm

# Setup the default mounts
#lxc.mount.auto = cgroup:mixed proc:mixed sys:mixed
lxc.mount.auto = cgroup:mixed proc:rw sys:mixed
lxc.mount.entry = /dev/snd dev/snd none bind,optional,create=dir
......
```
因为容器中需要对tty的完整控制，在host侧添加以下命令:     

```
$ sudo crontab -e
......
@reboot chmod 777 /dev/tty* && chmod 777 -R /dev/dri/ && chmod 777 /dev/fb0
```

### 2. zkfd
创建一个名为`zkfd`的lxc实例：    

```
# lxc-create -t local -n zkfdlxc -- -m /root/meta.tar.xz -f /root/zkfdlxc.tar.xz
Unpacking the rootfs

---
You just created an Ubuntu jammy amd64 (20241021_07:42) container.

To enable SSH, run: apt install openssh-server
No default root or user password are set by LXC.
```
手动添加透传设备规则:    

```
# vim /var/lib/lxc/zkfdlxc/config
......
lxc.mount.entry = /dev/fb0 dev/fb0 none bind,optional,create=file
lxc.mount.entry = /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry = /dev/dri/renderD128 dev/renderD128 none bind,optional,create=file
### allow tty8
lxc.mount.entry = /dev/tty7 dev/tty7 none bind,optional,create=file
lxc.mount.entry = /dev/tty8 dev/tty8 none bind,optional,create=file
lxc.mount.entry = /dev/tty0 dev/tty0 none bind,optional,create=file
#lxc.mount.entry = /dev/tty1 dev/tty1 none bind,optional,create=file
#lxc.mount.entry = /dev/tty2 dev/tty2 none bind,optional,create=file
#lxc.mount.entry = /dev/tty3 dev/tty3 none bind,optional,create=file
### allow all of the input
lxc.mount.entry = /dev/input dev/input none bind,optional,create=dir
### allow all of the snd
lxc.mount.entry = /dev/snd dev/snd none bind,optional,create=dir
```
更改lightdm:     

```
# vim /var/lib/lxc/zkfdlxc/rootfs/etc/lightdm/lightdm.conf 
......
[LightDM]
......
minimum-vt=8
......
```
更改events:    

```
# mv /var/lib/lxc/zkfdlxc/rootfs/etc/acpi/events /var/lib/lxc/zkfdlxc/rootfs/etc/acpi/events.bak
```
#### 2.1 鼠标输入
此时可以看到界面，但是无法用鼠标操作，需要进行修改:    

```
# lxc-attach -n zkfdlxc

cat >/usr/local/bin/preX-populate-input.sh  << __EOF__
#!/usr/bin/env bash

### Creates config file for X with all currently present input devices
#   after connecting new device restart X (systemctl restart lightdm)
######################################################################

cat >/etc/X11/xorg.conf.d/10-lxc-input.conf << _EOF_
Section "ServerFlags"
     Option "AutoAddDevices" "False"
EndSection
_EOF_

cd /dev/input
for input in event*
do
cat >> /etc/X11/xorg.conf.d/10-lxc-input.conf <<_EOF_
Section "InputDevice"
    Identifier "\$input"
    Option "Device" "/dev/input/\$input"
    Option "AutoServerLayout" "true"
    Driver "evdev"
EndSection
_EOF_
done
__EOF__

chmod +x /usr/local/bin/preX-populate-input.sh
mkdir -p /etc/systemd/system/lightdm.service.d
cat > /etc/systemd/system/lightdm.service.d/override.conf << __EOF__
[Service]
ExecStartPre=/bin/sh -c '/usr/local/bin/preX-populate-input.sh'
SupplementaryGroups=video render input audio tty
__EOF__
reboot
```
此时，鼠标应该是可以使用的状态。
#### 2.2 音频配置
安装测试软件:    

```
sudo apt install -y smplayer mplayer
```
添加：   

```
usermod -aG audio test
/usr/bin/pactl load-module module-alsa-card device_id=1 ; /usr/bin/pactl load-module module-alsa-card device_id=0
```

#### 2.3 快速创建记录
via:

```
vim /var/lib/lxc/zkfdlxc1/config 
cp preX-populate-input.sh /var/lib/lxc/zkfdlxc1/rootfs/usr/local/bin/
chmod 777 /var/lib/lxc/zkfdlxc1/rootfs/usr/local/bin/preX-populate-input.sh 
mkdir -p /var/lib/lxc/zkfdlxc1/rootfs/etc/systemd/system/lightdm.service.d
mkdir -p /var/lib/lxc/zkfdlxc1/rootfs/etc/X11/xorg.conf.d/
cp override.conf /var/lib/lxc/zkfdlxc1/rootfs/etc/systemd/system/lightdm.service.d
mv /var/lib/lxc/zkfdlxc1/rootfs/etc/acpi/events /var/lib/lxc/zkfdlxc1/rootfs/etc/acpi/events.back
```

### 3. kylin
创建:    

```
lxc-create -t local -n kylinlxc -- -m /root/meta.tar.xz -f /root/kylinlxc.tar.xz
```
仿照`2.3`创建相关目录并拷贝相关文件。   
创建成功后，需要手动安装:    

```
lxc-attach -n kylinlxc
# dhclient eth0
# apt update
# apt install -y xserver-xorg-input-evdev
```
