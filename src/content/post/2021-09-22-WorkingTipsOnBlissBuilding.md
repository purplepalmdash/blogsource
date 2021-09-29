+++
title= "WorkingTipsOnBlissBuilding"
date = "2021-09-22T10:11:05+08:00"
description = "WorkingTipsOnBlissBuilding"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Steps
Install ubuntu21.04 live server(90 Core, 16G Memory, 1T SSD)    

```
# sudo apt-get update -y
# sudo apt-get upgrade -y
# sudo apt-get install -y nethogs iotop
# sudo add-apt-repository ppa:openjdk/ppa
# sudo apt-get update && upgrade
# sudo apt-get install openjdk-8-jdk
# sudo apt-get install -y repo pypy-enum34
# sudo apt-get install git gnupg flex bison maven gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses-dev x11proto-core-dev libx11-dev lib32z1-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip squashfs-tools libssl-dev ninja-build lunzip syslinux syslinux-utils gettext genisoimage gettext bc xorriso libncurses5 xmlstarlet build-essential git imagemagick lib32readline-dev lib32z1-dev liblz4-tool libncurses5-dev libsdl1.2-dev libxml2 lzop pngcrush rsync schedtool  python3-mako libelf-dev
# sudo apt-get install -y shadowsocks-libev libevent-dev
Install sslocal configuration and redsocks for syncing source code
# git config --global user.email "xxx@gmail.com"
#  git config --global user.name "xxx"
```

Repo sync(This will take so long~long~long time, depending on your network speed):

```
$ mkdir ~/BuildBlissV14.x && cd ~/BuildBlissV14.x
$ repo init -u https://github.com/BlissRoms-x86/manifest.git -b r11-r36
$ repo sync -c --force-sync --no-tags --no-clone-bundle -j$(nproc --all) --optimized-fetch --prune
```

