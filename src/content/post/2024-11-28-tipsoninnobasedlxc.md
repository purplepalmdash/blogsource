+++
title= "tipsoninnobasedlxc"
date = "2024-11-28T06:36:04+08:00"
description = "tipsoninnobasedlxc"
keywords = ["Technology"]
categories = ["Technology"]
+++
config for lxc common:   

```
# cat /usr/share/lxc/config/common.conf
# Default configuration shared by all containers

# Setup the LXC devices in /dev/lxc/
lxc.tty.dir = lxc

# Allow for 1024 pseudo terminals
lxc.pty.max = 1024

# Setup 4 tty devices
lxc.tty.max = 4

# Drop some harmful capabilities
lxc.cap.drop = mac_admin mac_override sys_time sys_module sys_rawio

# Ensure hostname is changed on clone
lxc.hook.clone = /usr/share/lxc/hooks/clonehostname

# Default legacy cgroup configuration
#
# CGroup allowlist
lxc.cgroup.devices.deny = a
## Allow any mknod (but not reading/writing the node)
lxc.cgroup.devices.allow = c *:* m
lxc.cgroup.devices.allow = b *:* m
## Allow specific devices
### /dev/null
lxc.cgroup.devices.allow = c 1:3 rwm
### /dev/zero
lxc.cgroup.devices.allow = c 1:5 rwm
### /dev/full
lxc.cgroup.devices.allow = c 1:7 rwm
### /dev/tty
lxc.cgroup.devices.allow = c 5:0 rwm
### /dev/console
lxc.cgroup.devices.allow = c 5:1 rwm
### /dev/ptmx
lxc.cgroup.devices.allow = c 5:2 rwm
### /dev/random
lxc.cgroup.devices.allow = c 1:8 rwm
### /dev/urandom
lxc.cgroup.devices.allow = c 1:9 rwm
### /dev/pts/*
lxc.cgroup.devices.allow = c 136:* rwm
### fuse
lxc.cgroup.devices.allow = c 10:229 rwm
## graphics. /dev/dri
lxc.cgroup.devices.allow = c 226:0 rwm
lxc.cgroup.devices.allow = c 226:128 rwm
## graphics. /dev/fb0
lxc.cgroup.devices.allow = c 29:0 rwm
### tty0, tty1, tty7, tty8
lxc.cgroup.devices.allow = c 4:0 rwm
lxc.cgroup.devices.allow = c 4:1 rwm
lxc.cgroup.devices.allow = c 4:7 rwm
lxc.cgroup.devices.allow = c 4:8 rwm
### sound
lxc.cgroup.devices.allow = c 116:* rwm
### input
lxc.cgroup.devices.allow = c 13:* rwm

# Default unified cgroup configuration
#
# CGroup allowlist
lxc.cgroup2.devices.deny = a
## Allow any mknod (but not reading/writing the node)
lxc.cgroup2.devices.allow = c *:* m
lxc.cgroup2.devices.allow = b *:* m
## Allow specific devices
### /dev/null
lxc.cgroup2.devices.allow = c 1:3 rwm
### /dev/zero
lxc.cgroup2.devices.allow = c 1:5 rwm
### /dev/full
lxc.cgroup2.devices.allow = c 1:7 rwm
### /dev/tty
lxc.cgroup2.devices.allow = c 5:0 rwm
### /dev/console
lxc.cgroup2.devices.allow = c 5:1 rwm
### /dev/ptmx
lxc.cgroup2.devices.allow = c 5:2 rwm
### /dev/random
lxc.cgroup2.devices.allow = c 1:8 rwm
### /dev/urandom
lxc.cgroup2.devices.allow = c 1:9 rwm
### /dev/pts/*
lxc.cgroup2.devices.allow = c 136:* rwm
### fuse
lxc.cgroup2.devices.allow = c 10:229 rwm
## graphics. /dev/dri
lxc.cgroup2.devices.allow = c 226:0 rwm
lxc.cgroup2.devices.allow = c 226:128 rwm
## graphics. /dev/fb0
lxc.cgroup2.devices.allow = c 29:0 rwm
## tty0, 1, 7, 8
lxc.cgroup2.devices.allow = c 4:0 rwm
lxc.cgroup2.devices.allow = c 4:1 rwm
lxc.cgroup2.devices.allow = c 4:7 rwm
lxc.cgroup2.devices.allow = c 4:8 rwm
### sound
lxc.cgroup2.devices.allow = c 116:* rwm
### input
lxc.cgroup.devices.allow = c 13:* rwm

# Setup the default mounts
#lxc.mount.auto = cgroup:mixed proc:mixed sys:mixed
lxc.mount.auto = cgroup:mixed proc:rw sys:mixed
lxc.mount.entry = /sys/fs/fuse/connections sys/fs/fuse/connections none bind,optional 0 0

lxc.mount.entry = /dev/snd dev/snd none bind,optional,create=dir

# Block some syscalls which are not safe in privileged
# containers
lxc.seccomp.profile = /usr/share/lxc/config/common.seccomp

# Lastly, include all the configs from /usr/share/lxc/config/common.conf.d/
lxc.include = /usr/share/lxc/config/common.conf.d/

```
config added :    

```
lxc.mount.entry = /dev/fb0 dev/fb0 none bind,optional,create=file
lxc.mount.entry = /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry = /dev/dri/renderD128 dev/renderD128 none bind,optional,create=file
### allow tty8
lxc.mount.entry = /dev/tty7 dev/tty7 none bind,optional,create=file
lxc.mount.entry = /dev/tty8 dev/tty8 none bind,optional,create=file
lxc.mount.entry = /dev/tty0 dev/tty0 none bind,optional,create=file
#lxc.mount.entry = /dev/tty1 dev/tty1 none bind,optional,create=file
#lxc.mount.entry = /dev/tty2 dev/tty2 none bind,optional,create=file
#lxc.mount.entry = /dev/tty3 dev/tty3 none bind,optional,create=file
### allow all of the input
lxc.mount.entry = /dev/input dev/input none bind,optional,create=dir
### allow all of the snd
lxc.mount.entry = /dev/snd dev/snd none bind,optional,create=dir
```
The others are the same as previous ones.  
