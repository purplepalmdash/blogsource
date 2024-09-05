+++
title= "xrdpxorgOnUbuntu2204"
date = "2024-09-05T17:05:33+08:00"
description = "xrdpxorgOnUbuntu2204"
keywords = ["Technology"]
categories = ["Technology"]
+++
Install ubuntu 22.04 server, then :    

```
sudo apt update -y
sudo apt upgrade -y
sudo apt install -y ubuntu-desktop  nethogs 
```
Edit:    

```
$ sudo vim /etc/xrdp/xrdp.ini
...
allowed_users=anybody
...
```
Edit `nv_sock`:    

```
if [ ! -e /etc/modules-load.d/hv_sock.conf ]; then
	echo "hv_sock" > /etc/modules-load.d/hv_sock.conf
fi
```


Configure the policy xrdp session:    

```
cat > /etc/polkit-1/rules.d/02-allow-colord.rules <<EOF
polkit.addRule(function(action, subject) {
    if ((action.id == "org.freedesktop.color-manager.create-device" ||
         action.id == "org.freedesktop.color-manager.modify-profile" ||
         action.id == "org.freedesktop.color-manager.delete-device" ||
         action.id == "org.freedesktop.color-manager.create-profile" ||
         action.id == "org.freedesktop.color-manager.modify-profile" ||
         action.id == "org.freedesktop.color-manager.delete-profile") &&
        subject.isInGroup("users"))
    {
        return polkit.Result.YES;
    }
});
EOF
```

Create user and edit xinitrc:     

```
# cp /etc/X11/xinit/xinitrc ~/.xinitrc
```
Build xrdp and xorgxrdp:     

```
apt install -y git make autoconf libtool intltool pkg-config nasm xserver-xorg-dev libssl-dev libpam0g-dev libjpeg-dev libfuse-dev libopus-dev libmp3lame-dev libxfixes-dev libxrandr-dev libgbm-dev libepoxy-dev libegl1-mesa-dev libx264-dev
apt install -y libcap-dev libsndfile-dev libsndfile1-dev libspeex-dev libpulse-dev

apt install -y libfdk-aac-dev
apt install pulseaudio
apt install xserver-xorg

cd Code
git clone --branch devel --recursive https://github.com/neutrinolabs/xrdp.git
cd xrdp
./bootstrap
# Build with glamor explicitly enabled (does not appear to make a difference for core xrdp, but I kept this anyway)
./configure --enable-x264 --enable-glamor --enable-rfxcodec --enable-mp3lame --enable-fdkaac --enable-opus --enable-pixman --enable-fuse --enable-jpeg --enable-ipv6
# Control build statement (also works for me in Ubuntu 22.04, since it is xorgxrdp that actually links to glamor)
#./configure --enable-x264 --enable-rfxcodec --enable-mp3lame --enable-fdkaac --enable-opus --enable-pixman --enable-fuse --enable-jpeg --enable-ipv6
make -j4

make install

cd ~/Code
git clone --branch devel --recursive https://github.com/neutrinolabs/xorgxrdp.git
cd xorgxrdp

echo "-> Building xorgxrdp:"
./bootstrap
./configure --enable-glamor
make -j4

echo "-> Installing xorgxrdp:"
make install

systemctl enable xrdp
systemctl stop xrdp
systemctl start xrdp

sudo apt install gnome-tweaks -y
# Permission weirdness fix
sudo bash -c "cat >/etc/polkit-1/localauthority/50-local.d/45-allow.colord.pkla" <<EOF
[Allow Colord all Users]
Identity=unix-user:*
Action=org.freedesktop.color-manager.create-device;org.freedesktop.color-manager.create-profile;org.freedesktop.color-manager.delete-device;org.freedesktop.color-manager.delete-profile;org.freedesktop.color-manager.modify-device;org.freedesktop.color-manager.modify-profile
ResultAny=no
ResultInactive=no
ResultActive=yes
EOF
```

Configure:     

```
$ sudo vim /etc/X11/xrdp/xorg.conf
Section "Module"
.....
    Load "glamoregl"
....
```
Add user into render group:    

```
sudo usermod -aG render test1
sudo usermod -aG video test1
```

