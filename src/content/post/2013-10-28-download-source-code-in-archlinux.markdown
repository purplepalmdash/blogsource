---

comments: true
date: 2013-10-28T00:00:00Z
title: Download Source Code in ArchLinux
url: /2013/10/28/download-source-code-in-archlinux/
---

Install abs and base-devel

```
	$ pacman -S abs base-devel
```

As root, edit /etc/abs.conf to include your desired repositories:

```
	REPOS=(core extra community multilib !testing !community-testing !multilib-testing
	!staging !community-staging !gnome-unstable !kde-unstable)
```
As root, run "abs" to download the Arch Build System(ABS) tree.   
Find out which package the command belongs to, then use pacman -Ss to get the more detailed information, for example:

```
	[root@XXXyyy abs]# pacman -Qo audacious
	/usr/bin/audacious is owned by audacious 3.4.1-1
	[root@XXXyyy abs]# pacman -Ss audacious
	extra/audacious 3.4.1-1 [installed]
	    Lightweight, advanced audio player focused on audio quality
	extra/audacious-plugins 3.4.1-2 [installed]
	    Plugins for Audacious
```

Copy the description file from /var/abs/extra/ to your own place, then make the package :

```
	cp -r audacious* /media/x/code/
```

Then build the package, and also you can get the source files. 

```
	makepkg	
```
