+++
title= "WorkingTipsOnKata"
date = "2021-03-18T14:58:26+08:00"
description = "WorkingTipsOnKata"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Install & Configuration
Install kata on archlinux, first install snapd:    

```
$ yaourt snapd
$ sudo systemctl enable --now snapd.socket
```
Using snapd for installing kata:    

```
$ sudo snap install kata-containers --classic
```
Check the kata-container runtimes:    

```
$ kata-containers.runtime --version
kata-runtime  : 1.12.1
   commit   : b967088a667018b7468a9f93d48cb81650e0dfa4
   OCI specs: 1.0.1-dev
$ which kata-containers.runtime
/var/lib/snapd/snap/bin/kata-containers.runtime
```
Add the kata container runtime for docker-ce:    

```
$ sudo mkdir -p /etc/systemd/system/docker.service.d
$ sudo vim /etc/systemd/system/docker.service.d/kata-containers.conf 
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -D --add-runtime kata-runtime=/snap/kata-containers/current/usr/bin/kata-runtime
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```
Check the docker info:    

```
$ docker info | grep Runtime
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux kata-runtime runc
 Default Runtime: runc
```
### Testing
Run a busybox using kata-runtime:      

```
$ sudo docker run -ti --runtime kata-runtime busybox sh
```
Checking the docker hardware(qemu):       

```
/ # free -m
              total        used        free      shared  buff/cache   available
Mem:           1993          26        1965           0           2        1948
Swap:             0           0           0
/ # uname -a
Linux 172144f42ad4 5.4.60.container #1 SMP Wed Jan 20 17:43:09 UTC 2021 x86_64 GNU/Linux
```
Comparing to runc busybox:     

```
$ sudo docker run -it busybox /bin/sh
/ # free -m
              total        used        free      shared  buff/cache   available
Mem:          23932        3759       12883        1003        7289       18795
Swap:          2047           0        2047
/ # uname -a
Linux 7d484813ddd3 5.10.16-arch1-1 #1 SMP PREEMPT Sat, 13 Feb 2021 20:50:18 +0000 x86_64 GNU/Linux
```
Get the running qemu :     

```
# ps -ef | grep qemu
root      130733  130681  0 14:41 ?        00:00:03 /var/lib/snapd/snap/kata-containers/716/usr/bin/qemu-system-x86_64 -name sandbox-172144f42ad4130671d2f3282f84be7d33f17ec9f308234d9172162f6dac8a1f -uuid 07ebc86a-91a7-4180-accd-c9d1dbd3ac29 -machine pc,accel=kvm,kernel_irqchip,nvdimm -cpu host,pmu=off -qmp unix:/
.....
```

### Useful tips
Get the kata env:    

```
$ kata-containers.runtime kata-env
```
See if the system is ready for running kata:    

```
$ sudo kata-containers.runtime kata-check
```
