---
categories: ["null"]
comments: true
date: 2014-11-02T00:00:00Z
title: git under proxy
url: /2014/11/02/git-under-proxy/
---

This changes use the socks proxy for git.    
Install connect-proxy in aur:    

```
yaourt -S connect-proxy

```
Edit the proxy definition file:    

```
[Trusty@~]$ cat socks5.sh 
#!/bin/sh
connect -S 127.0.0.1:1394 "$@"

```
Now directly set the 

```
export GIT_PROXY_COMMAND=/home/Trusty/socks5.sh

```
Or change the proxy definition via:    

```
[core]
        gitproxy = /home/Trusty/socks5.sh

```
