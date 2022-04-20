+++
title= "WorkingTipsOnx11spice"
date = "2022-04-19T15:19:35+08:00"
description = "WorkingTipsOnx11spice"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Installation Steps
Install some necessary packages for building:    

```
$ sudo apt-get install -y build-essential autoconf xutils-dev libtool libgtk-3-dev libspice-server-dev build-essential
$ apt-cache search xcb | awk {'print $1'} | xargs -I % sudo apt-get install -y %
$ ./autogen.sh
$ ./configure --prefix=/usr
$ make && sudo make install
```
### Configuration
Copy the configuration file into the system configuration folder:    

```
$ sudo cp -r /usr/etc/xdg/x11spice /etc/xdg/
$ sudo vim /etc/xdg/x11spice/x11spice.conf 
...
listen=5900
disable-ticketing=true
allow-control=true
hide=true
display=:0
...
```
Login to the x session and run with:   

```
$ x11spice
```
View via:    

```
remote-viewer spice://192.168.xx.xx:5900 --spice-debug
```
### xserver-xspice
Install via:    

```
# apt install xserver-xspice ubuntu-desktop
```
Start a screen session via:    

```
$ sudo Xspice --password 123456 :0
```
In other session:    

```
$ DISPLAY=:0 gnome-session
OR
$ DISPLAY=:0 mate-session
```
view via:   

```
remote-viewer spice://192.168.xx.xx:5900 --spice-debug
```
