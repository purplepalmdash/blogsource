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
Install failed because it requires gitlab-shell 9.3.0 while the repository didn't provide this one.  Install gitlab-ce instead:    

```
# sudo apt-get purge gitlab
# sudo apt-get purge gitlab-common
# wget https://packages.gitlab.com/gpg.key
# sudo apt-key add gpg.key 
# sudo vim /etc/apt/sources.list
deb http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/debian buster main
# sudo apt-get update -y
# sudo apt-get install -y gitlab-ce
```
Configure the gitlab-ce:    

```
# sudo vim /etc/gitlab/gitlab.rb
external_url 'http://cicd.cicdforrong.ai'
# export LC_ALL=en_US.UTF-8
# export LANG=en_US.utf8
# sudo gitlab-ctl reconfigure
```
Configure the port(nginx/unicorn):   

```
# vi /etc/gitlab/gitlab.rb
nginx['listen_port'] = 82 #默认值即80端口 nginx['listen_port'] = nil
# vi /var/opt/gitlab/nginx/conf/gitlab-http.conf
listen *:82; #默认值listen *:80;
# vi /etc/gitlab/gitlab.rb
unicorn['port'] = 8082#原值unicorn['port'] = 8080
# vim /var/opt/gitlab/gitlab-rails/etc/unicorn.rb
listen "127.0.0.1:8082", :tcp_nopush => true
#原值listen "127.0.0.1:8080", :tcp_nopush => true
```
Reconfigure and restart the gitlab service:     

```
# gitlab-ctl reconfigure
# gitlab-ctl restart
```
### Visit gitlab
Edit the `/etc/hosts` for adding following items:    

```
# vim /etc/hosts
192.168.122.90	cicd.cicdforrong.ai
```
Now visit the `http://cicd.cicdforrong.ai` you could get the page for change username/password.    

### Install docker(for gitlab-runner)
Steps:    

```
#  sudo apt-get install     apt-transport-https     ca-certificates     curl     gnupg2     software-properties-common
#  curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
#  apt-key fingerprint 0EBFCD88
#  add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
#  apt-get update
#  apt-get install docker-ce docker-ce-cli containerd.io
```
