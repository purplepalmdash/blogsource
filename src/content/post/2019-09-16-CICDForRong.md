+++
title = "CICDForRong"
date = "2019-09-16T16:44:20+08:00"
description = "CICDForRong"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Steps
Download iso and install debian:   

```
# axel http://mirrors.163.com/debian-cd/10.1.0/amd64/iso-cd/debian-10.1.0-amd64-netinst.iso
# Create qemu virtual machine. 8-Core, 9G Memory
```
After installation:   

```
$ su root
# apt-get install -y vim sudo net-tools usermode
# sudo apt-get install -y gnupg2 gnupg1 gnupg
# vim /etc/apt/sources.list
deb http://mirrors.163.com/debian/ buster main
deb http://security.debian.org/debian-security buster/updates main
deb http://mirrors.163.com/debian/ buster-updates main
# apt-get update -y
# su -
# usermod -aG sudo dash
```
### Gitlab
Configure the repository file like following:      

```
# vim /etc/apt/sources.list
deb http://mirrors.163.com/debian/ buster main non-free contrib
deb http://security.debian.org/debian-security buster/updates main non-free contrib
deb http://mirrors.163.com/debian/ buster-updates main non-free contrib
deb http://mirrors.163.com/debian/ buster-backports main non-free contrib

### GitLab 12.0.8
deb http://fasttrack.debian.net/debian/ buster-fasttrack main contrib
deb http://fasttrack.debian.net/debian/ buster-backports main contrib 
deb https://deb.debian.org/debian buster-backports contrib main
# Eventually the packages in this repo will be moved to one of the previous two repos
deb https://people.debian.org/~praveen/gitlab buster-backports contrib main
```
signature:     

```
# wget https://people.debian.org/~praveen/gitlab/praveen.key.asc
# wget http://fasttrack.debian.net/fasttrack-archive-key.asc
# apt-key add praveen.key.asc
# apt-key add fasttrack-archive-key.asc
```
Install via:     

```
# apt -t buster-backports install gitlab
```


