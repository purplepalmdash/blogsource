---
categories: ["Linux"]
comments: true
date: 2013-12-17T00:00:00Z
title: Fastly search package infos in ArchLinux
url: /2013/12/17/fastly-search-package-infos-in-archlinux/
---

You can use pkgfile to view the metadata of the pacman files:

```
	pacman -Ss pkgfile
	extra/pkgfile 11-1 [installed]
	    a pacman .files metadata explorer

```
Usage:

```
	pkgfile ls

```
Then you will see "ls" belogns to which package. 

