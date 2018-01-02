---
categories: ["RaspberryPI", "Linux"]
comments: true
date: 2013-12-30T00:00:00Z
title: Use RaspberryPI as a file server
url: /2013/12/30/use-raspberrypi-as-a-file-server/
---

###Preparation
Mount the partition automatically, add following lines into the /etc/fstab:

```
	/dev/sda3	/media/ntfs    	ntfs-3g    permissions,locale=en_US.utf8  	  0	  2

```
I decide to use samba to share the files, so I have to install samba

```
	apt-get install samba

```
The samba server will start automatically, but we have to configure it to adapte to our situation.     
Add the configuration to the samba config file:

```
	/etc/samba/smb.conf
	[raspshare]
	   comment = raspberry PI Share
	   path = /media/ntfs
	   valid users = Trusty
	   public = no
	   writable = yes
	   printable = no
	   create mask = 0765

```
Restart the samba server

```
	/etc/init.d/samba restart

```
For using smbpasswd, you have to install samba-common-bin:

```
	apt-get install samba-common-bin

```
Add some users to the samba sharing:

```
	smbpasswd -a xxx

```
###Mount the shared partition in client
List all of the available samba shared items:

```
	$ smbclient -L 10.0.0.230 -U%

```
Then we can mount it, I add following command into the .bashrc, so everytime I enter 'mountraspsamba' is ok

```
	alias mountraspsamba='sudo mount -t cifs //10.0.0.230/raspshare/ -o user=xxxx,password=XXXX,workgroup=WORKGROUP /media/raspsamba'

```
