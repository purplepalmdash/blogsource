+++
date = "2017-07-02T12:33:39+08:00"
categories = ["Linux"]
keywords = ["Linux"]
description = "WorkingTipsOnDockerDeployment"
title = "WorkingTipsOnDockerDeployment"

+++
### 问题背景
针对某公司现有的混乱的发布、部署流程提出的一种基于Docker的应用程序部署方案。

### 架构
物理机+Docker运行环境+Docker-Compose编排+Docker镜像+Ansible自动化配置框架，达到`开箱即用`的快速部署目的。    

### 准备
#### Vagrantbox基础环境
Vagrantbox基于Ubuntu Xenial(16.04), 在其上安装了docker, docker-compose:    

```
# sudo apt-get update -y
# curl -sSL http://acs-public-mirror.oss-cn-hangzhou.aliyuncs.com/docker-engine/test/internet | sh
# sudo apt-get install docker-compose
```
由此构建的vagrant包含了docker运行所需要的最小条件，也就是基于最小条件，我们可以部署预定义好的docker服务。达到"开箱即用"的目的。    

可以通过`vagrant package`打包我们建立好的vagrant环境,
由此生成的`vagrant.box`可以用于新建环境的分发. 
#### 测试代码
基于`docker-haproxy-nginx`,
这个工程将建立一个nginx与haproxy协同作用的web集群，haproxy在本机的8080端口创建服务，将服务分别转发到后端的nginx1/nginx2/nginx3服务器，我们基于此测试代码构建我们的服务框架:    

```
# git clone git://github.com/kakakakakku/docker-haproxy-nginx.git
```
为模拟离线部署，手动pull回对应的docker镜像并保存之:    

```
# sudo docker pull haproxy:1.6.4-alpine
# sudo docker pull nginx:stable-alpine
# sudo docker save haproxy:1.6.4-alpine>haproxy.tar
# sudo docker save nginx:stable-alpine>nginx.tar
```
#### ansible部署脚本
ansible部署脚本用于自动执行环境配置，在准备好基础环境和测试代码、对应的docker镜像后，我们将对应撰写相应的ansible-playbook，
用于完成一键部署。    

### 手动部署流程
创建基于我们前面制作好的vagrantbox的虚拟机，给定一个新的IP地址，例如`192.168.33.201`:    

```
# mkdir testdockerdeployment
# cd testdockerdeployment
# vagrant init Xenial64DockerCompose
# vim Vagrantfile
  config.vm.network "private_network", ip: "192.168.33.201"
# vagrant up
# vagrant ssh
```
手动添加两个docker镜像:    

```
# docker load<haproxy.tar
# docker load<nginx.tar
```
开始服务:    

```
# docker-compose up
# ./requests.sh
```
### 与gitlab集成
在gitlab中创建一个公有的仓库, 用于存放自动化配置:    

![/images/2017_07_02_14_54_11_689x634.jpg](/images/2017_07_02_14_54_11_689x634.jpg)

在仓库下添加用于本地任务的`local.yml`:    

```
# vim local.yml
---
- hosts: 127.0.0.1

  tasks:

  - name: update docker compose
    shell: echo "update docker compose">>/tmp/docker.log
    notify: restart docker compose

  handlers:

  - name: restart docker compose
    shell: /home/vagrant/.local/bin/docker-compose -f /var/lib/ansible/local/docker-compose.yml stop && /home/vagrant/.local/bin/docker-compose -f /var/lib/ansible/local/docker-compose.yml build && /home/vagrant/.local/bin/docker-compose -f /var/lib/ansible/local/docker-compose.yml up -d
```
指定作用范围:    

```
[localhost]
127.0.0.1
```
通过ansible-pull来执行该仓库的定义如下:    

```
# root ansible-pull -i /var/lib/ansible/local/inventory.ini -d /var/lib/ansible/local -U http://root@192.168.33.2/root/centraladmin_docker.git -o
```
### 自动化
自动化主要通过crontab来执行。     

```
# vim /etc/cron.d/dockercompose-startup
@reboot /home/vagrant/.local/bin/docker-compose -f /var/lib/ansible/local/docker-compose.yml up -d
# vim /etc/cron.d/ansible-pull
*/3 * * * * root ansible-pull -i /var/lib/ansible/local/inventory.ini -d /var/lib/ansible/local -U http://root@192.168.33.2/root/centraladmin_docker.git -o >>/var/log/ansible-pull.log 2>&1

```

### 自动配置crontab
通过下面的ansible task来配置:    

```
# vim local_ansible_pull.yml
---

- hosts: 127.0.0.1
  sudo: yes
  #remote_user: root

  vars:

    # schedule is fed directly to cron
    schedule: '*/3 * * * *'

    # User to run ansible-pull as from cron
    cron_user: root

    # File that ansible will use for logs
    logfile: /var/log/ansible-pull.log

    # Directory to where repository will be cloned
    workdir: /var/lib/ansible/local

    # Repository to check out -- YOU MUST CHANGE THIS
    # repo must contain a local.yml file at top level
    repo_url: http://root@192.168.33.2/root/centraladmin_docker.git

  tasks:

    - name: Install ansible
      apt: pkg=ansible state=installed

    - name: Create local directory to work from
      file: path={{workdir}} state=directory owner=root group=root mode=0751

    - name: Create crontab entry to clone/pull git repository
      template: src=templates/etc_cron.d_ansible-pull.j2 dest=/etc/cron.d/ansible-pull owner=root group=root mode=0644

    - name: Create crontab entry to start docker-compose at startup
      template: src=templates/etc_cron.d_dockercompose.j2 dest=/etc/cron.d/dockercompose-start owner=root group=root mode=0644

    - name: Create logrotate entry for ansible-pull.log
      template: src=templates/etc_logrotate.d_ansible-pull.j2 dest=/etc/logrotate.d/ansible-pull owner=root group=root mode=0644
```

