---
categories: ["Virtualization"]
comments: true
date: 2015-04-28T00:00:00Z
title: Trouble Shooting On Juju's Local Deployment
url: /2015/04/28/trouble-shooting-on-jujus-local-deployment/
---

When deploying juju, after `juju bootstrap`, use juju ssh for login, it will hint me:    

```
$ juju ssh 1
......
Permission denied (publickey).

```
That could be solved by specify the id_rsa.pub key:     

```
$ ssh-keygen -t rsa -b 2048
$ juju bootstrap
$ juju bootstrap
$ juju deploy wordpress
$ juju deploy mysql
$ juju add-relation wordpress mysql
$ juju status
$ juju expose wordpress

```
By doing this you could make your juju deployment on local successfully.   
