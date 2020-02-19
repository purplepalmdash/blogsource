+++
title = "AlpineOffline"
date = "2018-04-07T08:54:40+08:00"
description = "AlpineOffline"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Background
I want to run `play-with-docker` offline, so I have to setup the whole offline
environment. In chapter 2 of `play-with-docker`, it requires building image
using alpine, its command `apk add nodejs` requires internet-connection. In
order to let the whole tutorial working offlinely, I have to setup an Alpine
offline repository and let it working.    

### Vagrant Env
The libvirt vagrant box is easy to download and run via following command:    

```
# vagrant init generic/alpine37
# vagrant up --provider=libvirt
# vagrant ssh
# cat /etc/issue
Welcome to Alpine Linux 3.7
Kernel \r on an \m (\l)
```
### Repository Server
This server could reach internet, so it could get packages, and generate the
installation repository.    

Take `nodejs` for example, I will create its offline installation reository in
this machine:   
 
```
#  apk add nodejs
```
This command will download all of the nodejs related package under
`/var/cache/apk`, then we could use these packages for installation:    

```
# apk index -vU --allow-untrusted -o /etc/apk/cache/APKINDEX.tar.gz /etc/apk/cache/*.apk
```
Using following scripts for generating symbolic `cheating` apks for
installation:    

```
#!/bin/bash
for f in $( cd /etc/apk/cache && ls *.apk ); do g=$(echo ${f:0:-13}.apk); cd /etc/apk/cache; ln -s $f $g; done
```
Your directory `/etc/apk/cache` will like following:    

```
# ls -l -h | more
total 98612
-rw-r--r--    1 root     root       14.4K Apr  6 08:31 APKINDEX.tar.gz
lrwxrwxrwx    1 root     root          28 Apr  6 08:32 abuild-3.1.0-r3.apk -> abuild-3.1.0-r3.e1614238.apk
-rw-r--r--    1 root     root       71.5K Apr  6 05:57 abuild-3.1.0-r3.e1614238.apk
lrwxrwxrwx    1 root     root          26 Apr  6 08:32 acct-6.6.4-r0.apk -> acct-6.6.4-r0.f2caf476.apk
-rw-r--r--    1 root     root       57.7K Mar 22 17:31 acct-6.6.4-r0.f2caf476.apk
-rw-r--r--    1 root     root        1.6K Mar 22 17:31 alpine-base-3.7.0-r0.6e79e3bb.apk
lrwxrwxrwx    1 root     root          33 Apr  6 08:32 alpine-base-3.7.0-r0.apk -> alpine-base-3.7.0-r0.6e79e3bb.apk
```
Using following commands for uploading repository to http server:    

```
scp -r /var/cache/apk/ xxx@192.168.122.1:/var/download/myapk/
```
In server `192.168.122.1`'s folder `/var/download/myapk`, do following
operation:    

```
# ln -s x86_64 noarch
```

### Client
In client, do following setting:    

```
# vim  /etc/apk/repositories 
    #https://dl-3.alpinelinux.org/alpine/v3.7/main
    #https://mirror.leaseweb.com/alpine/v3.7/main
    http://192.168.122.1/myapk
```
Update the repository via following command:    

```
# apk update --allow-untrusted
fetch http://192.168.122.1/myapk/x86_64/APKINDEX.tar.gz
OK: 110 distinct packages available
```
Install nodejs via following commands:    

```
# apk add nodejs --allow-untrusted
(1/8) Installing nodejs-npm (8.9.3-r1)
(2/8) Installing c-ares (1.13.0-r0)
(3/8) Installing libcrypto1.0 (1.0.2o-r0)
(4/8) Installing http-parser (2.7.1-r1)
(5/8) Installing libssl1.0 (1.0.2o-r0)
(6/8) Installing libstdc++ (6.4.0-r5)
(7/8) Installing libuv (1.17.0-r0)
(8/8) Installing nodejs (8.9.3-r1)
Executing busybox-1.27.2-r8.trigger
OK: 228 MiB in 82 packages
```
### TBD
You could add trusted signature to repository, but this will let you get the
signature firstly.    
