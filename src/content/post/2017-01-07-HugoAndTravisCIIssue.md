+++
categories = ["Linux"]
keywords = ["Linux"]
description = "Trouble Shooting for using travisCI for building hugo blog"
title = "Hugo And TravisCI Issue"
date = "2017-01-07T10:25:59+08:00"

+++
### Problem
In this morning when I get up and try to write something in my blog, I found
the blog won't upate. In travisCI website I got something very strange like
following picture shows:    

![/images/2017_01_07_10_29_39_562x249.jpg](/images/2017_01_07_10_29_39_562x249.jpg)    

Error info:    

```
Failed to normalize URL string. Returning in = "/"
```
### Reason
As discussed in this post:    

[https://discuss.gohugo.io/t/started-getting-failed-to-normalize-url-string-returning-in/5034](https://discuss.gohugo.io/t/started-getting-failed-to-normalize-url-string-returning-in/5034)    

This is because hugo now holds its own dependencies using govendor, you could
view from its repository:    

```
$ cd $GOPATH
$ cd src/github.com/spf13/hugo 
$ ls -l vendor 
total 16
-rw-r--r-- 1 dash root 14793 Jan  3 11:23 vendor.json
```
### Solution
Using govendor for syncing the dependencies, the modified .travis.yml is
listed as following:    

```
install:
    - go get -u -v github.com/kardianos/govendor
    - go get -u -v github.com/spf13/hugo
    - cd $GOPATH/src/github.com/spf13/hugo && govendor sync && go install
  

script:
    - cd $HOME/gopath/src/github.com/purplepalmdash/purplepalmdash.github.io && hugo
```
Save changes and commit it into github, the travis building will start again,
this time you won't failed.    
