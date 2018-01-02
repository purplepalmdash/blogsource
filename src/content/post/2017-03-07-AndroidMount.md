+++
date = "2017-03-07T23:28:15+08:00"
keywords = ["Linux"]
description = "Andoird mount in archlinux"
title = "Android Mount"
categories = ["Linux"]

+++
The steps in Ubuntu includes:    

```
$ sudo pacman -S pcmanfm mtpfs gvfs-mtp gvfs-gphoto2 libmtp
```

Edit the fuse configuration via:    

```
$ sudo vim /etc/fuse.conf
user_allow_other
```
Mount the equipment to ~/mnt:    

```
$ mkdir -p ~/mnt
$ mtpfs -o allow_other ~/mnt
```
Now open pcmanfm you will see the attached new android equipment.    

Or now if you insert the android phone, the pcmanfm will automatically mount
the equipment's file system.    
