+++
title= "LxcBasedDesktop"
date = "2023-10-16T15:11:27+08:00"
description = "LxcBasedDesktop"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Create lxc instance  

![/images/2023_10_16_15_11_38_711x258.jpg](/images/2023_10_16_15_11_38_711x258.jpg)

![/images/2023_10_16_15_11_50_503x177.jpg](/images/2023_10_16_15_11_50_503x177.jpg)

![/images/2023_10_16_15_12_00_430x157.jpg](/images/2023_10_16_15_12_00_430x157.jpg)

![/images/2023_10_16_15_12_17_396x134.jpg](/images/2023_10_16_15_12_17_396x134.jpg)

![/images/2023_10_16_15_12_24_417x157.jpg](/images/2023_10_16_15_12_24_417x157.jpg)

![/images/2023_10_16_15_12_33_627x280.jpg](/images/2023_10_16_15_12_33_627x280.jpg)

![/images/2023_10_16_15_12_46_717x496.jpg](/images/2023_10_16_15_12_46_717x496.jpg)

Configuration file for pve 100:    

```
# cat /etc/pve/lxc/100.conf 
arch: amd64
cores: 4
features: nesting=1
hostname: ubuntu2204
memory: 8192
net0: name=eth0,bridge=vmbr0,firewall=1,hwaddr=DA:8A:5D:E2:3D:1F,ip=dhcp,type=veth
ostype: ubuntu
rootfs: local-lvm:vm-100-disk-0,size=40G
swap: 512
unprivileged: 1
lxc.cgroup2.devices.allow: c 226:0 rwm
lxc.cgroup2.devices.allow: c 226:128 rwm
lxc.cgroup2.devices.allow: c 29:0 rwm
lxc.mount.entry: /dev/fb0 dev/fb0 none bind,optional,create=file
lxc.mount.entry: /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry: /dev/dri/renderD128 dev/renderD128 none bind,optional,create=file
lxc.cgroup2.devices.allow: c 4:7 rwm
lxc.mount.entry: /dev/tty7 dev/tty7 none bind,optional,create=file
lxc.cgroup2.devices.allow: c 13:* rwm
lxc.mount.entry: /dev/input dev/input none bind,optional,create=dir
lxc.cgroup2.devices.allow: c 116:* rwm
lxc.mount.entry: /dev/snd dev/snd none bind,optional,create=dir
lxc.idmap: u 0 100000 65536
lxc.idmap: g 0 100000 5
lxc.idmap: g 5 5 1
lxc.idmap: g 6 100006 23
lxc.idmap: g 29 29 1
lxc.idmap: g 30 100030 14
lxc.idmap: g 44 44 1
lxc.idmap: g 45 100045 60
lxc.idmap: g 105 101 1
lxc.idmap: g 106 100106 2
lxc.idmap: g 108 103 1
lxc.idmap: g 109 100109 65427
```
Start the machine and ssh into it.   

### Configuration in lxc

```
apt update -y && apt upgrade -y
apt install -y curl sudo gnupg
apt install -y va-driver-all ocl-icd-libopencl1
apt-get install -y lightdm
echo "/usr/sbin/lightdm" > /etc/X11/default-display-manager
apt-get install -y kodi
apt install -y kodi-peripheral-joystick
cat <<EOF >/usr/share/xsessions/kodi-alsa.desktop
[Desktop Entry]
Name=Kodi-alsa
Comment=This session will start Kodi media center with alsa support
Exec=env AE_SINK=ALSA kodi-standalone
TryExec=env AE_SINK=ALSA kodi-standalone
Type=Application
EOF
useradd -d /home/kodi -m kodi &>/dev/null
gpasswd -a kodi audio &>/dev/null
gpasswd -a kodi video &>/dev/null
gpasswd -a kodi render &>/dev/null
groupadd -r autologin &>/dev/null
gpasswd -a kodi autologin &>/dev/null
gpasswd -a kodi input &>/dev/null

cat <<EOF >/usr/share/xsessions/kodi-alsa.desktop
[Desktop Entry]
Name=Kodi-alsa
Comment=This session will start Kodi media center with alsa support
Exec=env AE_SINK=ALSA kodi-standalone
TryExec=env AE_SINK=ALSA kodi-standalone
Type=Application
EOF

cat <<EOF >/etc/lightdm/lightdm.conf.d/autologin-kodi.conf
[Seat:*]
autologin-user=kodi
autologin-session=kodi-alsa
EOF

apt-get install -y xserver-xorg-input-evdev
mkdir -p /etc/X11/xorg.conf.d

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

ln -fs /lib/systemd/system/lightdm.service /etc/systemd/system/display-manager.service
systemctl daemon-reload
systemctl start lightdm
ln -fs /lib/systemd/system/lightdm.service /etc/systemd/system/display-manager.service
```
Change to xfce4:    

```
apt install -y xfce4
root@ubuntu2204:~# cat /etc/lightdm/lightdm.conf.d/autologin-kodi.conf 
[Seat:*]
autologin-user=kodi
autologin-session=xfce4-alsa
#autologin-session=kodi-alsa
root@ubuntu2204:~# cat /usr/share/xsessions/xfce4-alsa.desktop 
[Desktop Entry]
Name=xfce4-alsa
Comment=This session will start xfce4 with alsa support
Exec=env AE_SINK=ALSA startxfce4
TryExec=env AE_SINK=ALSA startxfce4
Type=Application
```
Change to dde:    

```
add-apt-repository ppa:ubuntudde-dev/stable
apt install ubuntudde-dde

```
