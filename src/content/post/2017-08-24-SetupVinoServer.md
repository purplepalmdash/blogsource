+++
title = "SetupVinoServer"
date = "2017-08-24T15:42:39+08:00"
description = "setup vino server in CentOS7"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Background
For enable rdp server, but I installed vino server for vncserver, faint.   

### CentOS Configuration
Install Gnome Desktop.   

Enable root autologin(This is pretty dangerous):    

```
# vim /etc/gdm/custom.conf 

[daemon]
AutomaticLoginEnable=true
AutomaticLogin=username
```
So next time you will login to the desktop automatically.    

### Vino Server
Vino will be automatically installed for gnome desktop ,config it via:    

```
# gsetting set org.gnome.Vino require-encryption false
```
Also setting up the password.    

![/images/2017_08_24_15_49_18_565x466.jpg](/images/2017_08_24_15_49_18_565x466.jpg)

Now reboot the machine you could enjoy the vncserver.   

### xrdp
Install xrdp via epel:   

```
# yum install -y xrdp
# systemctl enable xrdp.service
# systemctl start xrdp.service
# echo "mate-session" > ~/.Xclients
# chmod +x ~/.Xclients
# sudo systemctl restart xrdp.service
```
Then using some rdp clients for connecting to desktop.   

### Finding minimal packages

```
# /bin/bash
for i in `cat xrdp.txt`
do
	find_or_not=`find /media/sdb/centos73-isorepo/Packages/ | grep $i`
	if  [ $? == 1 ]
	then
		echo $i
	fi
done
```
