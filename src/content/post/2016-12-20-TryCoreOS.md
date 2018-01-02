+++
categories = ["Virtualization"]
date = "2016-12-20T15:35:29+08:00"
description = "Try CoreOS in libvirt"
keywords = ["Virtualization"]
title = "TryCoreOS"

+++
### Preparation
Create image file via:     

```
$ sudo mkdir corecluster
$ cd corecluster
$ qemu-img create -f qcow2 coreos1.qcow2 30G
$ qemu-img create -f qcow2 coreos2.qcow2 30G
$ qemu-img create -f qcow2 coreos3.qcow2 30G
```
Create a network named `172.17.8.1/24`, dhcp disabled.     

Since the vnet interface is occupied via virtualbox, switches to virtualbox
installation.  

Two Ethernet cards:    

![/images/2016_12_20_17_36_44_697x413.jpg](/images/2016_12_20_17_36_44_697x413.jpg)    

Start from CD:    

![/images/2016_12_20_17_37_26_678x304.jpg](/images/2016_12_20_17_37_26_678x304.jpg)    

### Installation File Preparation
For saving time, we use local installation repository, download the
installation images via:    

```
$ $ git clone https://github.com/coreos/coreos-baremetal
# Make a copy of example files
$ cp -R coreos-baremetal/examples .
# Download the CoreOS image assets referenced in the target profile.
$ ./coreos-baremetal/scripts/get-coreos stable 1185.5.0 ./examples/assets
```
Then Copy installation image and sig file into the webserver:    

```
$ cd /YourWebServerDirectory/1185.5.0 
$ ls
coreos_production_image.bin.bz2  coreos_production_image.bin.bz2.sig
```

### YAML Definition
Refers to `coreos-vagrant` project:    

```
$ git clone https://github.com/coreos/coreos-vagrant.git`
$ cat user-data
```

The configuration of the yaml is listed as following:    

First get the token via:    

```
$ curl https://discovery.etcd.io/new?size=3
https://discovery.etcd.io/xxxxxxxxxxxxxxxxxxxxxxxxx
```

```
#cloud-config
hostname: coreos1 
coreos:
  etcd2:
    # generate a new token for each unique cluster from https://discovery.etcd.io/new?size=3
    # specify the initial size of your cluster with ?size=X
    discovery: https://discovery.etcd.io/xxxxxxxxxxxxxxxxxxx
    advertise-client-urls: http://172.17.8.201:2379,http://172.17.8.201:4001
    initial-advertise-peer-urls: http://172.17.8.201:2380
    # listen on both the official ports and the legacy ports
    # legacy ports can be omitted if your application doesn't depend on them
    listen-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001
    listen-peer-urls: http://172.17.8.201:2380,http://172.17.8.201:7001
  fleet:
    public-ip: "172.17.8.201"
  flannel:
    interface: "172.17.8.201"
  units:
    - name: etcd2.service
      command: start
    - name: fleet.service
      command: start
    - name: flanneld.service
      drop-ins:
      - name: 50-network-config.conf
        content: |
          [Service]
          ExecStartPre=/usr/bin/etcdctl set /coreos.com/network/config '{ "Network": "10.1.0.0/16" }'
      command: start
    - name: static.network
      content: |
        [Match]
        Name=enp0s8
        [Network]
        Address=172.17.8.201/24
        Gateway=172.17.8.1
        DNS=172.17.8.1
    - name: docker-tcp.socket
      command: start
      enable: true
      content: |
        [Unit]
        Description=Docker Socket for the API
  
        [Socket]
        ListenStream=2375
        Service=docker.service
        BindIPv6Only=both
  
        [Install]
        WantedBy=sockets.target
users:  
  - name: core
    ssh-authorized-keys: 
      - ssh-rsa "ADD ME"
  - groups:
      - sudo
      - docker
```
For coreos2 node, simply replace the ip address from `172.17.8.201` to `172.17.8.202`, etc.    

### Installation
Install it via:    

```
# coreos-install -d /dev/sda -b http://YourWebServer -c ./YourYamlFile.yaml -v
```

After a while, the installation will finish.   

Repeat this step in 3 nodes.   

### Verification
Login into any of the core machine in cluster:    

```
core@coreos1 ~ $ etcdctl cluster-health
member f934a5ba1eca1ea is healthy: got healthy result from http://172.17.8.201:2379
member 23798f79754d53b7 is healthy: got healthy result from http://172.17.8.203:2379
member 5acdd34e67ade1d7 is healthy: got healthy result from http://172.17.8.202:2379
cluster is healthy
``` 

