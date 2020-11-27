+++
title= "WorkingTipsOnSVNClientInDocker"
date = "2020-11-27T18:03:04+08:00"
description = "WorkingTipsOnSVNClientInDocker"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Prepare docker images
pull ubuntu latest image:    

```
# sudo docker run -it ubuntu:latest /bin/bash
```
Install svn and related items in docker:    

```
# sudo apt-get update -y
# sudo apt-get install -y subversion language-pack-zh*
# localedef -c -f UTF-8 -i zh_CN zh_CN.utf8
```
Commit the docker images:    

```
# sudo docker commit 67ef51b2783e ubuntusvn:latest
```
### Run docker in Intranet
Run svn commit in docker. 

```
# docker run -it -v /media/xxxxxx:/mnt ubuntusvn:latest
#  export LC_ALL=zh_CN.UTF_8
# cd /mnt/cloud/
# svn add xfsiso/ubuntu-18.04.5-server-amd64-auto-xfs.iso
# svn ci -m "Added ubuntu18.04.5 server iso" --username=xxxxxx --password=xxxxxx123
```
