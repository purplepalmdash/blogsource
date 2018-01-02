---

comments: true
date: 2013-10-25T00:00:00Z
title: Using wicd with Awesome
url: /2013/10/25/using-wicd-with-awesome/
---

###Install packages
Install the following packages

```
	$ sudo pacman -S wicd
	$ sudo pacman -S wicd-gtk
	$ sudo pacman -S notification-daemon
	$ sudo pacman -S python2-notify
```

###Configure
Add your account to users group

```
	# gpasswd -a USERNAME users
```

Start Wicd as System Service:

```
	$ systemctl start wicd.service
```

Automatically start service at boot-up:

```
	$ systemctl enable wicd.service
```

Add the wicd-client as tray in the rc.lua file

```
	$ wicd-client --tray
```


