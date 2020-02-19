+++
title = "WorkingTipsOnPlayWithDockerUbuntu"
date = "2018-08-14T15:53:36+08:00"
description = "WorkingTipsOnPlayWithDockerUbuntu"
keywords = ["Linux"]
categories = ["Technology"]
+++
### dnsmasq for ubuntu
systemd-resolved will listen on 53 port, disable it via:    

```
# vim /etc/systemd/resolved.conf
DNSStubListener=no
# systemctl enable dnsmasq
```
Disable the system-resolved.service:    

```
# systemctl disable systemd-resolved.service
# systemctl stop systemd-resolved.service
# echo nameserver 192.168.0.15>/etc/resolv.conf
# chattr -e /etc/resolv.conf
# chattr +i /etc/resolv.conf
# ufw disable
# docker swarm leave
# docker swarm init
```


### registry proxy issue
Enable the registry proxy will slow down the registry cached image download
speed, solved it via:    

```
# vim config.yml

....
#proxy:
#	remoteurl: https://registry-1.docker.io
```
Then restart the docker registry service(the docker instance), the pulling
speed will greatly reduced.    

### dind Registry
registry mirror should point to a real ip, rather than `172.17.0.1`, another
word, you could not use inner docker bridge address for registry usage.    

### Golang
Go could be directly migration from the old iso distribution.   

### cubic issue
Alway use a new directory for customize your new iso distribution!!!!!!    


