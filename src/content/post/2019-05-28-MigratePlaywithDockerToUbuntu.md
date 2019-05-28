+++
title = "MigratePWDtoUbuntu"
date = "2019-05-28T08:47:00+08:00"
description = "MigratePWDtoUbuntu"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Steps
Install Ubuntu 18.04.02(Using Rong iso).    
Configure IP/hostname.    
Install docker-ce, docker-compose.   
Load all of the docker images.    
Change the dnsmasq.    
Enable the service(playwithdocker/playwithdockerblog).    
Install golang and re-run go(the previously go directory could be re-use)   
Run `docker swarm init` before you really run the playwithdocker.   
Add items into dnsmasq (192.192.189.115/192.192.189.115)
Modification on code:    

```
$ playwithdockerblog, 
192.192.189.114->115
$ playwithdocker(/root/go/src/github.com/playwithdocker/playwithdocker/xxx.go
114->115
```

### dnsmasq
For conclicting with libvirt's dnsmasq, do following steps:    

```
# vim /etc/dnsmasq.conf
listen-address=192.192.189.127
bind-interfaces
# systemctl restart dnsmasq
```
Thus we would remove the dnsmasq funcitonality from the exisiting
playwithdocker nodes(previously we run dnsmasq on 192.192.189.114 rather than
in 192.192.189.127.   

### Result
Now using browser for viewing `192.192.189.115`, then you could see a new
running playwithdocker blog.   
