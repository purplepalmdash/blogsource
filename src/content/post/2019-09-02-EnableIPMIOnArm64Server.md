+++
title = "EnableIPMIMonitoringOnArm64Server"
date = "2019-09-02T09:14:53+08:00"
description = "EnableIPMIMonitoringOnArm64Server"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Add iso repository
Use installation iso as repository:     

```
# mount -t iso9660 -o loop ubuntu180402_arm64.iso /mnt
# vim /etc/apt/sources.list
deb [trusted=yes] file:///mnt bionic main contrib
# apt-get update -y
# apt-cache search | grep ipmi
freeipmi-common - GNU implementation of the IPMI protocol - common files
freeipmi-tools - GNU implementation of the IPMI protocol - tools
libfreeipmi16 - GNU IPMI - libraries
libipmiconsole2 - GNU IPMI - Serial-over-Lan library
libipmidetect0 - GNU IPMI - IPMI node detection library
maas - "Metal as a Service" is a physical cloud and IPAM
libopenipmi0 - Intelligent Platform Management Interface - runtime
openipmi - Intelligent Platform Management Interface (for servers)
```
### ipmi
Install two packages:     

```
# apt-get install -y openipmi freeipmi-tools
```

### Build netdata
Using a docker instance on vps for building netdata:     

```
# docker run -it ubuntu:bionic-20190424 /bin/bash
# cat /etc/issue

# apt-get update -y
# apt-get install -y vim build-essential

```
