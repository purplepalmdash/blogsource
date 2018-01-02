+++
description = "Working tips on gitlab ci"
title = "WorkingTipsOnGitLabCI"
categories = ["Linux"]
keywords = ["Linux"]
date = "2017-06-14T08:30:37+08:00"

+++
### 背景
设置GitLabCI的流程。    

使用两台虚拟机节点来实现GitLab服务器/GitLabCI节点, CI工作节点。    

硬件配置:    
GitLab服务器及CI节点: 2核3G内存。    
CI工作节点: 2核2G内存。

运行系统: CentOS 7.3 X86_64
### GitLab节点配置
#### gitlab-ce
配置gitlab-ce库及安装gitlab-ce:    

```
# vim /etc/yum.repos.d/gitlab-ce.repo
[gitlab-ce] 
name=gitlab-ce 
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7 
repo_gpgcheck=0 
gpgcheck=0 
enabled=1 
gpgkey=https://packages.gitlab.com/gpg.key
# yum makecache && yum install -y gitlab-ce
```
配置gitlab-ce并使能服务:    

```
# gitlab-ctl reconfigure
# firewall-cmd --permanent --add-service=http
```
如果你的机器没有开启firewalld, 则firewalld这条命令则无需键入。    
#### gitlab-ci-multi-runner
配置gitlab-ci-multi-runner库:    

```
# vim /etc/yum.repos.d/gitlab-ce.repo
[gitlab-ci-multi-runner]
name=gitlab-ci-multi-runner
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ci-multi-runner/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
```
安装gitlab-ci：    

```
# yum makecache && yum install -y gitlab-ci-multi-runner
```
现在访问`http://192.168.33.2`，则可进入到gitlab的配置页面，设置root及用户密码.    

### GitLab Ci Runner节点
安装上面的`gitlab-ci-multi-runner`即可    

安装完以后，配置如下:    

```
# gitlab-ci-multi-runner register
```

配置例子如下:    

```
# cat /etc/gitlab-runner/config.toml
concurrent = 4
check_interval = 0

[[runners]]
  name = "shellrunner"
  url = "http://192.168.33.2/ci"
  token = "0e39a08e63d4c4355e9dae4e3784ab"
  executor = "shell"
  [runners.cache]

[[runners]]
  name = "ourtester"
  url = "http://192.168.33.2/ci"
  token = "605a6900c4805efa1e52629391ac32"
  executor = "shell"
  [runners.cache]

[[runners]]
  name = "dockerrunner"
  url = "http://192.168.33.2/ci"
  token = "f8f6de41ddfe4a2dcf1e792df6c1c4"
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "node:4.5.0"
    privileged = false
    disable_cache = false
    volumes = ["/cache","/root/m2:/root/.m2"]
    pull_policy = "if-not-present"
    shm_size = 0
  [runners.cache]
```

因为我们需要使用docker,所以在该节点上要安装最新版的docker:    

```
# curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/internet | sh -
# docker version
Client:
 Version:      17.05.0-ce
......
```
### 配置
在project下可以找到的runner配置信息如下:    

![/images/2017_06_15_21_50_49_1045x642.jpg](/images/2017_06_15_21_50_49_1045x642.jpg)

注意 `Specify the following URL during the Runner setup: http://192.168.33.2/ci `
和`Use the following registration token during setup: 2DpeAyEAeNxg1fmxVezv `

这两个和我们在上面注册runner的时候所需要填的是一样的。    

创建完Project `firstproject`后，可以使用以下命令拷贝到本地:    

```
# git clone git@192.168.33.2:root/firstproject.git
```

### 项目配置
创建一个`.gitlab-cimyml`文件于`firstproject`的目录下:    

```
image: "my3dlib:latest"
stages:
  - dependencies
  - build
  - test
  - deploy

ceres-build:
  stage: dependencies
  script:
    - export I3D_CERES_VERSION=1.11.0
    - wget --quiet http://192.168.33.1/ceres-solver-1.11.0.tar.gz
    - mkdir ceres-source ceres-build ceres-install
    - tar xvfz ceres-solver-1.11.0.tar.gz -C ceres-source --strip-components=1
    - cmake -Bceres-build -Hceres-source
    - make -j$(nproc) -C ceres-build
    - make -C ceres-build install DESTDIR=../ceres-install
    - bash .gitlab_build_files/build_ceres_debian_pkg.sh
  artifacts:
    paths:
    - i3d-ceres_*_amd64.deb
  tags:
    - dockerrunner
    #- linux,debian-jessie
cores-test:
  stage: test
  script:
    - cat /proc/cpuinfo && df -h
cores-deploy:
  stage: deploy
  script:
    - ssh-keyscan -H 192.168.33.1 >> ~/.ssh/known_hosts
    - sshpass -p xxxxx scp -P 22 i3d-ceres_*_amd64.deb dash@192.168.33.1:/tmp/
```
我们在这里还使用到了`build_ceres_debian_pkg.sh`文件，这个文件是用于编译deb包的，内容如下:    

```
# mkdir -p .gitlab_build_files
# vim .gitlab_build_files/build_ceres_debian_pkg.sh
#! /usr/bin/env bash

fpm \
-t deb \
-s dir \
-C ceres-install \
--name "i3d-ceres" \
--version 1.11.0 \
--license "BSD" \
--vendor "ICG TU Graz" \
--category "devel" \
--architecture "amd64" \
--maintainer "Aerial Vision Group <aerial@icg.tugraz.at>" \
--url "https://aerial.icg.tugraz.at/" \
--description "Compiled Ceres solver for i3d library" \
--depends cmake \
--depends libatlas-dev \
--depends libatlas-base-dev \
--depends libblas-dev \
--depends libeigen3-dev \
--depends libgoogle-glog-dev \
--depends liblapack-dev \
--depends libsuitesparse-dev \
--verbose \
.
```
现在，每一次提交的更改，都将触发自动编译过程。    

![/images/2017_06_15_21_59_09_1275x453.jpg](/images/2017_06_15_21_59_09_1275x453.jpg)
### docker镜像制作
这里我们使用了my3dlib这个名称的docker镜像，镜像的制作过程如下:    

```
# vim Dockerfile
FROM buildpack-deps:jessie
MAINTAINER Alexander Skiba <alexander.skiba@icg.tugraz.at>

ENV DEBIAN_FRONTEND noninteractive

RUN echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" > /etc/apt/sources.list && echo "deb http://mirrors.163.com/debian/ jessie-updates main non-free contrib" >> /etc/apt/sources.list && echo "deb http://mirrors.163.com/debian/ jessie-backports main non-free contrib" >> /etc/apt/sources.list && echo "deb http://mirrors.163.com/debian-security/ jessie/updates main non-free contrib" >> /etc/apt/sources.list

RUN sleep 10

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    cmake \
    freeglut3 \
    freeglut3-dev \
    gcc \
    git \
    g++ \
    libatlas-dev \
    libatlas-base-dev \
    libboost-all-dev \
    libblas-dev \
    libcgal-dev \
    libdevil-dev \
    libeigen3-dev \
    libexiv2-dev \
    libglew-dev \
    libgoogle-glog-dev \
    liblapack-dev \
    liblas-dev \
    liblas-c-dev \
    libpcl-dev \
    libproj-dev \
    libprotobuf-dev \
    libqglviewer-dev \
    libsuitesparse-dev \
    libtclap-dev \
    libtinyxml-dev \
    mlocate \
    ruby \
    ruby-dev \
    unzip \
    wget \
    sshpass \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/* \
  && gem install --no-rdoc --no-ri fpm
# docker build -t my3dlib .
```
经过编译以后，我们将得到my3dlib这个镜像，用于编译我们后面所需要的deb文件。
### artifacts
`artifacts`记录了每次我们自动编译所生成的内容:   

![/images/2017_06_15_22_02_18_1293x452.jpg](/images/2017_06_15_22_02_18_1293x452.jpg)

解压后即可得到deb文件。   

### external_url
这里记载了如果不配置实际IP地址将导致的错误.    

You should configure the `external_url` to your specified IP address, or you
won't successfully pulling codes into your working nodes:    

```
$ vim /etc/gitlab/gitlab.rb
external_url 'http://192.168.33.2'
$ gitlab-ctl reconfigure
$ gitlab-ctl restart
```
Now you could successfully pulling codes into your working nodes.    

### 下一步
研究gitlab ci与kubernetes的集成。    
