+++
categories = ["Linux"]
date = "2016-07-21T21:12:27+08:00"
description = "Build Packer And Its Plugins"
keywords = []
title = "编译Packer及其插件"

+++
### Install Go
In ArchLinux, do following for installing and configurating go:    

```
$ vim ~/.bash_profile
export GOPATH=$HOME/go
export PATH=$PATH:$GOPATH/bin
```
Exit the terminal and relogin again, now you could verify your GOPATH and golang System Path.    

Install go in ArchLinux via:    

```
$ sudo pacman -S go
```

### Installing Dev Packer

