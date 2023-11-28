+++
title= "LXDBasedDesktopOnIDV"
date = "2023-11-28T09:55:16+08:00"
description = "LXDBasedDesktopOnIDV"
keywords = ["Technology"]
categories = ["Technology"]
+++
Upgrade snapd and install lxd:     

```
apt install snapd
snap install lxd
lxd init
```
Create the first instance:    

```
# lxc launch ubuntu:22.04
Creating the instance
Instance name is: proper-frog                 
Starting proper-frog
```
