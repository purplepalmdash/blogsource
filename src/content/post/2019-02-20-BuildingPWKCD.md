+++
title = "BuildingPWKCD"
date = "2019-02-20T16:14:53+08:00"
description = "BuildingPWKCD"
keywords = ["Linux"]
categories = ["Linux"]
+++
### MakeISO Server
Configure via:    

```
# apt-get update -y 
# apt-get install -y vim openssh-server
```
Install cubic via:    

```
# apt-add-repository ppa:cubic-wizard/release
# apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 6494C6D6997C215E
# apt-get update -y && apt-get install -y cubic
```
### cubic make iso
Start cubic via:    

![/images/2019_02_20_16_20_41_429x333.jpg](/images/2019_02_20_16_20_41_429x333.jpg)

Create the iso project folder:    

```
# mkdir ~/isoproject
```

Slect the original disk image to customize:    

![/images/2019_02_20_16_28_24_516x413.jpg](/images/2019_02_20_16_28_24_516x413.jpg)

Cubic will copy the content from the origin folder to remote folder, this will
takes for some time:    

![/images/2019_02_20_16_29_11_495x357.jpg](/images/2019_02_20_16_29_11_495x357.jpg)

In chroot terminal you could custom the cd:    

![/images/2019_02_20_16_32_14_516x207.jpg](/images/2019_02_20_16_32_14_516x207.jpg)

### CD customize
Install docker/docker-compose

```
# vim /etc/apt/sources.list(Changes to 163.com)
# apt-get install -y python-pip && pip install docker-compose
# ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
# apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# apt-key fingerprint 0EBFCD88
# add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
# apt-get install docker-ce docker-ce-cli containerd.io
# docker version
```
The current Version is :    

![/images/2019_02_20_16_52_47_505x133.jpg](/images/2019_02_20_16_52_47_505x133.jpg)

Cause we are under chroot, we don't have server running.    

We fetch the pwk ready machine's `/var/lib/docker`, transfer them into our
chroot env:    

![/images/2019_02_20_17_06_27_549x122.jpg](/images/2019_02_20_17_06_27_549x122.jpg)

We also need golang for running the pwk environment:    

```
# apt-get install -y golang
# vim /root/.bashrc

```
Transfer the golang environment from the pwk ready machine:    

```
# ls /root
 go/ Code/
```
systemd file:    

```
root@test-Standard-PC-Q35-ICH9-2009:/etc/systemd/system# cat mynginx.service 
[Unit]
Description=mynginx
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker start -a docker-nginx
ExecStop=/usr/bin/docker stop -t 2 docker-nginx

[Install]
WantedBy=multi-user.target

root@test-Standard-PC-Q35-ICH9-2009:/etc/systemd/system# cat playwithdockerblog.service 
[Unit]
Description=playwithdockerblog
After=docker.service
Requires=docker.service

[Service]
ExecStart=/usr/bin/docker-compose -f /root/Code/play-with-kubernetes.github.io/docker-compose.yml up -d

[Install]
WantedBy=multi-user.target
root@test-Standard-PC-Q35-ICH9-2009:/etc/systemd/system# cat playwithdocker.service 
[Unit]
Description=playwithdocker
After=docker.service
Requires=docker.service

[Service]
Environment=GOPATH=/root/go/
WorkingDirectory=/root/go/src/github.com/play-with-docker/play-with-docker
Type=idle
# Remove old container items
ExecStartPre=/usr/bin/docker-compose -f /root/go/src/github.com/play-with-docker/play-with-docker/docker-compose.yml down
# Compose up
ExecStart=/usr/bin/docker-compose -f /root/go/src/github.com/play-with-docker/play-with-docker/docker-compose.yml up -d

[Install]
WantedBy=multi-user.target
```

ToBeContinue, later I will change the static website content, and k8s image
offline, then do the iso-build. 

