+++
title= "AndroidInAosp11AVDWorkingTips"
date = "2021-12-23T14:44:45+08:00"
description = "AndroidInAosp11AVDWorkingTips"
keywords = ["Technology"]
categories = ["Technology"]
+++
### 0. 目的
本文目的是为了记录如何在基于aosp11的avd开启redroid容器实例。    

### 1. 准备aosp源码并运行avd
下载tsinghua的repo用于同步代码, 由于repo需要使用python3来同步代码，故需要安装`python-is-python3`包:    

```
$ curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
$ chmod a+x repo
$ sudo apt-get install -y python-is-python3
```
repo的运行过程中会尝试访问官方的git源更新自己，指定使用tuna的镜像源进行更新:    

```
$ vim ~/.bashrc
export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
```
创建目录并开始同步代码(具体时间取决于网络状态), 如果需要同步别的分支的源码，可以参考这里(`https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds`):     

```
$ mkdir aosp11
$ cd aosp11
$ repo init -u https://mirrors.tuna.tsinghua.edu.cn/git/AOSP/platform/manifest -b android-11.0.0_r48
$ repo sync -j8
```

