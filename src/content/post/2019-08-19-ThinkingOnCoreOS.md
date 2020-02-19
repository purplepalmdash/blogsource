+++
title = "ThinkingOnCoreOS"
date = "2019-08-19T11:21:39+08:00"
description = "ThinkingOnCoreOS"
keywords = ["Linux"]
categories = ["Technology"]
+++
### AIM
To deploy kubespray-deployed kubernetes offlinely.     

### Problems
1. Operating system?    
2. How to run ansible?    
3. How to run ssl configurations?    
4. How to setup dns name server and pretend the docker.io/quay.io/gcr.io?    
5. Scale up/down the cluster?    
6. Netdata?    
7. docker-compose based systemd configuration?    

### Steps
#### 1. Environments
vagrant based coreos OS.    
From the kubespray's Vagrantfile, judge the coreos box file:    

![/images/2019_08_19_11_32_29_952x262.jpg](/images/2019_08_19_11_32_29_952x262.jpg)

Then download the latest coreos vagrant box via:     

```
# wget https://stable.release.core-os.net/amd64-usr/2135.6.0/coreos_production_vagrant.box
```
#### 2. dns nameserver
Run dns nameserver in docker.  

```
# sudo docker run --name dnsmasq -d -p 53:53/udp -p 5380:8080 -v /home/core/dnsmasq.conf:/etc/dnsmasq.conf --log-opt "max-size=100m"  -e "HTTP_USER=foo"  -e "HTTP_PASS=bar"  --restart always  jpillora/dnsmasq
# cat dnsmasq.conf 
#dnsmasq config, for a complete example, see:
#  http://oss.segetech.com/intra/srv/dnsmasq.conf
#log all dns queries
log-queries
#dont use hosts nameservers
no-resolv
#use cloudflare as default nameservers, prefer 1^4
server=1.0.0.1
server=1.1.1.1
strict-order
#serve all .company queries using a specific nameserver
server=/company/10.0.0.1
#explicitly define host-ip mappings
address=/myhost.company/10.0.0.2
address=/portus.xxxxx.com/10.142.18.191
address=/gcr.io/10.142.18.191
address=/k8s.gcr.io/10.142.18.191
address=/quay.io/10.142.18.191
address=/elastic.co/10.142.18.191
address=/docker.elastic.co/10.142.18.191
address=/docker.io/10.142.18.191
address=/registry-1.docker.io/10.142.18.191
```
Later we will configure dns for `/etc/resolv.conf` in coreos.   

#### 3. dns client
For configurating the client's dns name server, do:    

```
#  cat /etc/systemd/resolved.conf 
  #  This file is part of systemd.
  #
  #  systemd is free software; you can redistribute it and/or modify it
  #  under the terms of the GNU Lesser General Public License as published by
  #  the Free Software Foundation; either version 2.1 of the License, or
  #  (at your option) any later version.
  #
  # Entries in this file show the compile time defaults.
  # You can change settings by editing this file.
  # Defaults can be restored by simply deleting this file.
  #
  # See resolved.conf(5) for details
  
  [Resolve]
  DNS=10.142.18.191
# sudo systemctl restart systemd-resolved
```
#### 4. auto-index file server
systemd file could be run, just as usual.   

#### 5. ansible version 
Using virtualenv for installing ansible 2.7:    

```
#  sudo apt install virtualenv
# virtualenv -p python3 env 
# source env/bin/activate
# which pip
/media/sda/coreos_kubespray/Rong/env/bin/pip
# pip install ansible==2.7.9
# which ansible
/media/sda/coreos_kubespray/Rong/env/bin/ansible
# ansible --version
ansible 2.7.9
  config file = /media/sda/coreos_kubespray/Rong/ansible.cfg
# pip install netaddr
```

### Disks
mounting the esdata disk via:    

```
# vim /etc/systemd/system/media-esdata.mount 
[Unit]
Description=Mount cinder volume
Before=docker.service

[Mount]
What=/dev/vdb1
Where=/media/esdata
Options=defaults,noatime,noexec

[Install]
WantedBy=docker.service
# systemctl enable media-esdata.mount
```
Then creating the filesystem on `/dev/vdb1`, your mounting point will be ok.   


