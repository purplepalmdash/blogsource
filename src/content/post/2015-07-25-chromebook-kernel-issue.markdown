---
categories: ["Linux"]
comments: true
date: 2015-07-25T11:34:48Z
title: Chromebook Kernel Issue
url: /2015/07/25/chromebook-kernel-issue/
---

### Issues
Chromebook could not support:   
* bluetooth LAN
* NFS    
* ETC    

So I re-compile the Chromebook kernel for soving these issue.   

### Kernel Version

```
# uname -r
3.10.18
```

### Get The SourceCode

```
# git clone --branch v3.10.18 https://chromium.googlesource.com/chromiumos/third_party/kernel
```
