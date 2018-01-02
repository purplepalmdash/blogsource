---
categories: ["virtualization"]
comments: true
date: 2016-04-22T19:39:58Z
title: rsync the vault.centos.com
url: /2016/04/22/rsync-the-vault-dot-centos-dot-com/
---

For I want to do some configuration workings on old distribution of CentOS, I have to
use lots of materials which from vault.centos.com, following are the steps for syncing
them.    

First, rsync in `vault.centos.com` is closed, thus we have to choose
`http://archive.kernel.org/centos/`.    

### Rsync Scripts
Refers to:     
[https://www.totalnetsolutions.net/2013/10/02/setting-up-a-corporate-yum-repository-mirror-for-bandwidth-and-staged-update-management/](https://www.totalnetsolutions.net/2013/10/02/setting-up-a-corporate-yum-repository-mirror-for-bandwidth-and-staged-update-management/)    

### Make Repository
[https://wiki.centos.org/HowTos/CreateLocalMirror](https://wiki.centos.org/HowTos/CreateLocalMirror)    
