+++
title = "Building k8s-dns-node-cache-arm64"
date = "2019-07-03T14:41:02+08:00"
description = "Buidlingtips"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Steps
Clone the source code from github:    

```
# git clone https://github.com/kubernetes/dns.git
# git checkout tags/1.15.1 -b local_1.15.1
```
Modify the source code:      

```
# vim Makefile
BINARIES := \
    node-cache
#    e2e \
#    ginkgo \
#    sidecar-e2e


CONTAINER_BINARIES := \
    node-cache

ARCH ?= arm64
```
Now building via:    

```
# make build
# make containers
```
Will get error, now copy the generated node-cache file into destination directory:     

```
# cp ./.go/bin/node-cache bin/arm64/
```
Edit the dnsmasq's Makefile via:     

```
# vim images/dnsmasq/Makefile
```

![/images/2019_07_03_14_47_54_635x612.jpg](/images/2019_07_03_14_47_54_635x612.jpg)

make will also get error, manually compile the dnsmasq:     


```
# cd ./images/dnsmasq/_output/arm64/dnsmasq-2.78
# make
# cp src/dnsmasq ../docker
```
Now go the project root directory and make containers:     

```
# make containers
# docker images  | grep dns
staging-k8s.gcr.io/k8s-dns-dnsmasq-arm64      1.15.1-dirty         fbb04ccb60e6        About an hour ago   3.63MB
staging-k8s.gcr.io/k8s-dns-node-cache-arm64   1.15.1-dirty         bf6131745b5e        About an hour ago   71.9MB
```
Now replace the dns-node-cache default to our own build-out version we could enable node-cache working on arm64.    

