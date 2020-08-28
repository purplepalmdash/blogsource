+++
title= "OfflineArm64DesktopEnvSetup"
date = "2020-08-27T17:19:40+08:00"
description = "OfflineArm64DesktopEnvSetup"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Download via docker
Run a docker instance via:    

```
$ sudo docker run -v /mnt:/mnt -it ubuntu:focal-20200115 /bin/bash
```

In docker instance, do following:   

```
rm -f /etc/apt/apt.conf.d/docker-clean
sed -i 's/ports.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
cd /mnt
apt-get -d -o dir::cache=`pwd` -o Debug::NoLocking=1 install xubuntu-desktop xubuntu-core chromium-browser  firefox xrdp virt-manager ubuntu-wallpapers xubuntu-community-wallpapers xubuntu-community-wallpapers-focal xubuntu-wallpapers lxd lxc
apt-get install -y snapd
snap download lxd
snap download chromium

```
### Transfer
Transfer these packages into the offline environments, and do following:    

```
# cd ~/pkgs
# dpkg-scanpackages . /dev/null | gzip -9c > Packages.gz
```
### Install
In a ubuntu base environment, do following:    

```
# vim /etc/apt/sources.list
deb [trusted=yes] file:///home/test/focal/ ./
# apt-get update -y
# apt-get install -y  xubuntu-desktop xubuntu-core  firefox xrdp virt-manager ubuntu-wallpapers xubuntu-community-wallpapers xubuntu-community-wallpapers-focal xubuntu-wallpapers
```

### Configure xrdp
Configure xrdp via:   

```
$ sudo systemctl status xrdp
$ sudo adduser xrdp ssl-cert  
$ sudo systemctl restart xrdp
$ sudo ufw disable
$ echo xfce4-session >~/.xsession
$ sudo vim /etc/xrdp/startwm.sh
#!/bin/sh

if [ -r /etc/default/locale ]; then
  . /etc/default/locale
  export LANG LANGUAGE
fi

startxfce4
$ sudo systemctl restart xrdp
```
So now you could use xfce4 as your remote desktop to linux. 

### snapd installation
In docker run:    

```
# sudo apt-get install -y snapd
# snap download lxd
# snap download chromium
# snap download gtk-common-themes
# snap download core
# snap download core18
```
Install sequence:    

```
# snap install  (core/core18/gtk-common-themes/lxd/chromium)
chromium_1253.snap  core18_1888.snap  core_9806.snap                 gtk-common-themes_1506.snap  lxd_16946.snap
chromium_1253.assert  core18_1888.assert  core_9806.assert  gtk-common-themes_1506.assert  lxd_16946.assert   
```

### Updated
xrdp configuration:    

```
#!/bin/sh -e

# Install XRDP.
sudo apt install -y xrdp
sudo sed -e 's/^new_cursors=true/new_cursors=false/g' \
     -i /etc/xrdp/xrdp.ini
sudo systemctl enable xrdp
sudo systemctl restart xrdp

# Load Ubuntu config.
echo "xfce4-session" > ~/.xsession
D=/usr/share/xfce4:/usr/share/xubuntu:/usr/local/share
D=${D}:/usr/share:/var/lib/snapd/desktop:/usr/share
cat <<EOF > ~/.xsessionrc
export XDG_SESSION_DESKTOP=xubuntu
export XDG_DATA_DIRS=${D}
export XDG_CONFIG_DIRS=/etc/xdg/xdg-xubuntu:/etc/xdg:/etc/xdg
EOF

# Disable light-locker for avoiding error.
sudo cp /usr/bin/light-locker /usr/bin/light-locker.orig
cat <<EOF | sudo tee /usr/bin/light-locker
#!/bin/sh

# The light-locker uses XDG_SESSION_PATH provided by lightdm.
if [ ! -z "\${XDG_SESSION_PATH}" ]; then
  /usr/bin/light-locker.orig
else
  # Disable light-locker in XRDP.
  true
fi
EOF
sudo chmod a+x /usr/bin/light-locker
```

### Final script
Final script is listed as following:    

```
#!/bin/bash
sudo cp -r focal /home/test/
echo 'deb [trusted=yes] file:///home/test/focal/ ./'|sudo tee -a /etc/apt/sources.list
sudo apt-get update -y
sudo apt-get install -y xubuntu-desktop xubuntu-core  firefox xrdp virt-manager ubuntu-wallpapers xubuntu-community-wallpapers xubuntu-community-wallpapers-focal xubuntu-wallpapers 
sudo adduser xrdp ssl-cert  
sudo systemctl restart xrdp
sudo ufw disable

sudo sed -e 's/^new_cursors=true/new_cursors=false/g' \
	     -i /etc/xrdp/xrdp.ini
sudo systemctl enable xrdp
sudo systemctl restart xrdp

# Load Ubuntu config.
echo "xfce4-session" > ~/.xsession
D=/usr/share/xfce4:/usr/share/xubuntu:/usr/local/share
D=${D}:/usr/share:/var/lib/snapd/desktop:/usr/share
cat <<EOF > ~/.xsessionrc
export XDG_SESSION_DESKTOP=xubuntu
export XDG_DATA_DIRS=${D}
export XDG_CONFIG_DIRS=/etc/xdg/xdg-xubuntu:/etc/xdg:/etc/xdg
EOF

# Disable light-locker for avoiding error.
sudo cp /usr/bin/light-locker /usr/bin/light-locker.orig
cat <<EOF | sudo tee /usr/bin/light-locker
#!/bin/sh

# The light-locker uses XDG_SESSION_PATH provided by lightdm.
if [ ! -z "\${XDG_SESSION_PATH}" ]; then
  /usr/bin/light-locker.orig
else
  # Disable light-locker in XRDP.
  true
fi
EOF
sudo chmod a+x /usr/bin/light-locker

# Install snapd
sudo snap ack snap/chromium_1253.assert
sudo snap ack snap/core18_1888.assert
sudo snap ack snap/core_9806.assert
sudo snap ack snap/gtk-common-themes_1506.assert
sudo snap ack snap/lxd_16946.assert

sudo snap install snap/core_9806.snap
sudo snap install snap/core18_1888.snap
sudo snap install snap/gtk-common-themes_1506.snap
sudo snap install snap/chromium_1253.snap
#sudo snap install snap/lxd_16946.snap

```
New user will be added if you want to create new session. 
