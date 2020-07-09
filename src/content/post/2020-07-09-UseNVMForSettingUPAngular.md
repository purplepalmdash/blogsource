+++
title= "UseNVMForSettingUPAngular"
date = "2020-07-09T09:13:21+08:00"
description = "UseNVMForSettingUPAngular"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Install NVM
Install nvm via:    

```
$ yaourt nvm
```
Then 

```
$ echo 'source /usr/share/nvm/init-nvm.sh' >> ~/.zshrc
$ which nvm
...
$ nvm install v10.21.0
$ node -v
v10.21.0
```

### Setup Env
Setup the angular proxy variables:    

```
# cat proxy.conf.json 
{
  "/api": {
    "target": "http://10.137.149.4",
    "secure": false
  },
  "/admin": {
    "target": "http://10.137.149.4",
    "secure": false
  },
  "/static": {
    "target": "http://10.137.149.4",
    "secure": false
  },
  "/ws": {
    "target": "http://10.137.149.4",
    "secure": false,
    "ws": true
  }
}
```
Install packages via:     

```
# SASS_BINARY_SITE=https://npm.taobao.org/mirrors/node-sass/ npm install node-sass@4.10.0
# npm --registry=https://registry.npm.taobao.org --disturl=https://npm.taobao.org/dist install
# sudo cnpm install -g @angular/cli
# sudo cnpm install -g @angular-devkit/build-angular/
# ng serve --proxy-config proxy.conf.json
```
After building you will have the debug environment for changing the code and view the result. 

