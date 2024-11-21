+++
title= "rebuildkernelforfc40"
date = "2024-11-21T14:59:51+08:00"
description = "rebuildkernelforfc40"
keywords = ["Technology"]
categories = ["Technology"]
+++
安装编译内核所需要的所有依赖:     

```
sudo dnf install fedpkg
git config --global http.proxy 'socks5://192.168.1.6:21080'
fedpkg clone -a kernel
cd kernel
sudo dnf builddep kernel.spec
```
FEdora dist-git内核包:    

```
mkdir fc_dist-git
cd fc_dist-git
git clone https://src.fedoraproject.org/rpms/kernel.git
```
根据发行版的版本号，切换到对应的分支:    

```
root@localhost:~/Code/fc_dist-git# cat /etc/redhat-release 
Fedora release 40 (Forty)
root@localhost:~/Code/fc_dist-git# cd kernel/
root@localhost:~/Code/fc_dist-git/kernel# git switch f40
分支 'f40' 设置为跟踪 'origin/f40'。
切换到一个新分支 'f40'
```
为了防止与现有的内核版本冲突，设置一个自定义的buildid, 为了加速编译，最好可以全局fanqiang:    

```
# vim kernel.spec
...
%define buildid .fucktyy
# fedpkg local
...
```

![/images/20241121_150632_x.jpg](/images/20241121_150632_x.jpg)

这里需要注意，之前安装过较低版本的手动编译的pahole, 需要恢复之前的:     

```
rm -f /usr/lib/libdwarves* /usr/lib64/libdwarves*
yum reinstall libdwarves1 dwarves
pahole --version
v1.26
```
