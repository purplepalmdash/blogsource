+++
categories = ["Technology"]
date = "2020-05-15T11:31:21+08:00"
description = "DockerImageSize"
keywords = ["Technology"]
title= "DockerImageSize"

+++
### 原生build/save
采用原生的build/save得到的结果:    

```
# docker build -t rong/core:v1.17.5 . && docker save -o rongcore.tar rong/core:v1.17.5
# ls -l -h rongcore.tar
-rw------- 1 root root 1.4G May 12 14:45 rongcore.tar

```

### Dockerfile更改
Dockerfile中添加 `RUN touch /tmp/requirements/abc` 一行，这样会触发新的build， 从21行起，21行前则沿用以前的层.    

![/images/2020_05_15_11_31_50_416x363.jpg](/images/2020_05_15_11_31_50_416x363.jpg)

开启 docker的 `--experimental=true` 选项(ArchLinux为例,不同操作系统版本可能不一样):    

```
#  /etc vim systemd/system/multi-user.target.wants/docker.service 
.....
ExecStart=/usr/bin/dockerd -H fd:// --experimental=true
....
#  /etc systemctl daemon-reload
#  /etc systemctl restart docker
```

开启编译:    

```
# docker build -t rong/core:v1.17.5 --squash .   
```

存储增量文件:    

```
# d-save-last rong/core:v1.17.5 -o /mnt6/v2.tar
Running dind-save container...
Running docker save...

Cleaning up...
# ls -l -h /mnt6/v2.tar 
-rw------- 1 root root 899M May 15 12:47 /mnt6/v2.tar
```

### 加载
加载时load v2.tar时，只加载经过改动的层:    

```
# docker load<v2.tar 
4beb03d58ef7: Loading layer [==================================================>]    942MB/942MB
The image rong/core:v1.17.5 already exists, renaming the old one with ID sha256:0a0de68c5f49fb7faf63a90719f10dd7749283344a06a73e9ddbc94a81377a8f to empty string
Loaded image: rong/core:v1.17.5

```
