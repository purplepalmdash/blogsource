+++
title = "WorkingTipsOnPlayWithDocker"
date = "2018-04-07T10:31:27+08:00"
description = "WorkingTipsOnPlayWithDocker"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Environment
#### Qemu Image
Qemu image preparation:    

```
# mkdir offline-play-with-docker
# cd offline-play-with-docker 
# qemu-img create -f qcow2 offline-play-with-docker.qcow2 200G
Formatting 'offline-play-with-docker.qcow2', fmt=qcow2 size=214748364800 cluster_size=65536 lazy_refcounts=off refcount_bits=16
```
#### Virt-manager Networking
Network Name:    

![/images/2018_04_07_10_33_34_346x179.jpg](/images/2018_04_07_10_33_34_346x179.jpg)
Definition of IPV4:    

![/images/2018_04_07_10_34_29_338x431.jpg](/images/2018_04_07_10_34_29_338x431.jpg)

Warning:   

![/images/2018_04_07_10_34_50_505x137.jpg](/images/2018_04_07_10_34_50_505x137.jpg)

Isolation(Could be adjust to isolated):    

![/images/2018_04_07_10_35_35_386x231.jpg](/images/2018_04_07_10_35_35_386x231.jpg)

Create vm and specify vmworks:    

![/images/2018_04_07_13_35_07_540x268.jpg](/images/2018_04_07_13_35_07_540x268.jpg)

Install CentOS 7.4, partition like following:    

![/images/2018_04_07_13_38_50_484x224.jpg](/images/2018_04_07_13_38_50_484x224.jpg)

![/images/2018_04_07_13_39_10_650x209.jpg](/images/2018_04_07_13_39_10_650x209.jpg)

### System
Set the hostname via:    

![/images/2018_04_07_14_35_11_543x279.jpg](/images/2018_04_07_14_35_11_543x279.jpg)

Install mate desktop(for debugging purpose):    

```
# wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# yum --enablerepo=epel -y groups install "MATE Desktop"
```
Install some tools:    

```
# yum install -y vim wget python-pip gcc git nethogs
# pip install shadowsocks
# pip inststall --upgrade pip
# yum install -y libevent-devel
# build the redsocks for crossing the gfw!!!
```
Now you could cross the firewall for installing the go or other
staffs(crossing the gfw WILL let everything more smoothly).    

Install docker-ce:    

```
#  yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
# yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
# yum install -y docker-ce
```
Install docker-compose via pip:    

```
# pip install docker-compose
```
Start and enable docker:    

```
# systemctl enable docker
# systemctl start docker
# docker swarm init
```
You have to run `docker swarm init`, or you won't login into the
play-with-docker.    

Install golang:    

```
# yum install -y golang
# mkdir ~/go
# vim ~/.bashrc
export GOPATH=/root/go
export PATH=$PATH:$GOPATH/bin
```

Create the directory and clone the source code:   

```
# mkdir ~/Code
# cd Code
# git clone https://github.com/play-with-docker/play-with-docker.git
```
Build the `play-with-go`:    

```
# go get -u github.com/golang/dep/cmd/dep
# which dep
/root/go/bin/dep
# cd ~/Code/play-with-docker/cd ~/Code/play-with-docker/
# go get -v -d -t ./...
# cd /root/go/src/github.com/play-with-docker/play-with-docker
# dep ensure
```
### Fixed IP
Fixed IP address then we could manually build the dind adjusting to this IP
address:    

![/images/2018_04_07_15_03_06_595x287.jpg](/images/2018_04_07_15_03_06_595x287.jpg)


### Local Registry
In order to work offline, we have to use local repository.    

```
# mkdir ~/data
# cd ~/data
# docker run -it --rm --entrypoint cat registry:2 /etc/docker/registry/config.yml > config.yml
# vim config.yml
proxy:
      remoteurl: https://registry-1.docker.io
# mdkir ~/data/data
# docker run -d --restart=always -p 5000:5000 --name docker-registry-proxy-2 -v /root/data/config.yml:/etc/docker/registry/config.yml -v /root/data/data:/var/lib/registry registry:2
```
Now examine the docker registry running:    

```
# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
0ea38c808e8d        registry:2          "/entrypoint.sh /etc…"   2 seconds ago       Up 1 second         0.0.0.0:5000->5000/tcp   docker-registry-proxy-2
```

### Build dind
dind is `docker in docker`, which is a docker image for running docker
instance inside docker, we build it manually:    

```
# vim
/root/go/src/github.com/play-with-docker/play-with-docker/dockerfiles/dind/daemon.json
    "insecure-registries": ["https://192.192.189.114:5000"],
# vim
/root/go/src/github.com/play-with-docker/play-with-docker/dockerfiles/dind/Dockerfile
......
    /usr/sbin/sshd -o PermitRootLogin=yes -o PrintMotd=no 2>/dev/null && \
    dockerd --registry-mirror=http://192.192.189.114:5000 &>/docker.log 
......
```
Build the image:    

```
# docker build -t franela/dind:latest .
# docker images | grep dind
franela/dind        latest              7832d23a42c7        About a minute ago   439MB
docker              stable-dind         d303f49c92a7        2 weeks ago          147MB
```
This dind image could use local registry, so we only need to sync once, then
we could let it running really offlinely.   
### Local play-with-docker
Change the source code:    

```
# cd /root/go/src/github.com/play-with-docker/play-with-docker
# vim handlers/bootstrap.go +64
- return false
+ return true
```

![/images/2018_04_07_15_32_59_766x271.jpg](/images/2018_04_07_15_32_59_766x271.jpg)

Run `play-with-docker`:    

```
# cd /root/go/src/github.com/play-with-docker/play-with-docker
# docker-compose up
```
Then use a browser to access this website:    

![/images/2018_04_07_16_08_09_749x759.jpg](/images/2018_04_07_16_08_09_749x759.jpg)

Examine the registry now:    

![/images/2018_04_07_16_09_00_655x229.jpg](/images/2018_04_07_16_09_00_655x229.jpg)

Then in host terminal, examine the downloaded registry cache:    

```
curl http://192.192.189.114:5000/v2/_catalog
{"repositories":["library/alpine","library/ubuntu"]}
```

### play-with-docker classroom
Clone the repository:    

```
# cd ~/Code
# git clone https://github.com/play-with-docker/play-with-docker.github.io.git
# cd play-with-docker.github.io/
# vim _config.yml
pwdurl: http://192.192.189.114

# mkdir _site
# groupadd jekyll
# useradd jekyll -m -g jekyll
# chown jekyll:jekyll -R .
# docker-compose up
```

Now open the browser and see the result:   

![/images/2018_04_07_16_49_34_794x532.jpg](/images/2018_04_07_16_49_34_794x532.jpg)


