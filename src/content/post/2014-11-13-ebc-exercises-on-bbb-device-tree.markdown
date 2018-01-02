---
categories: ["embedded"]
comments: true
date: 2014-11-13T00:00:00Z
title: EBC Exercises on BBB - Device Tree
url: /2014/11/13/ebc-exercises-on-bbb-device-tree/
---

### Install js wrapped bonescript
For using the javascript wrapped library, first we have to install following packages:    

```
$ sudo apt-get install nodejs npm nodejs-legacy
$ npm config set proxy http://1xx.xx.xx.xxx:2xxx
$ npm config set https-proxy http://1xx.xx.xx.xx:2xxx
$ npm install bonescript

```
Because my network environment is under the firewall, so I need to set the proxy, if you directly connect to internet, you could ignore the `npm config set` commands.    



