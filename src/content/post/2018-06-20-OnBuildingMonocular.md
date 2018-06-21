+++
title = "OnBuildMonocular"
date = "2018-06-20T17:18:50+08:00"
description = "OnBuildMonocular"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Building
Ubuntu 18.04(192.192.189.114), install following packages:    

```
# apt-get install -y nodejs npm
# git clone https://github.com/kubernetes-helm/monocular.git
# cd monocular/src/ui
# npm config set registry http://registry.npm.taobao.org/
# npm install
# npm install -g yarn
# make docker-build
```
Then you will get the `bitnami/monocular-ui:latest` docker image. 
