+++
title = "HarborArm64Issue"
date = "2020-01-13T14:11:04+08:00"
description = "HarborArm64Issue"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Issue
harbor-log instance launched first, but it complains:    

```
You are required to change your password immediately (password expired)
```
This is because we build the container images only have 90 days limitation.   

### Solution
Rebuild all of the harbor images:    

```
root@node:~/harbor# cat harbor-core/Dockerfile 
FROM f9e2034f3a6d
COPY login.defs /etc/login.defs
COPY shadow /etc/shadow
root@node:~/harbor# cat harbor-core/shadow 
root:x:18074:0:99999:7:::
bin:x:18074:0:99999:7:::
daemon:x:18074:0:99999:7:::
messagebus:x:18074:0:99999:7:::
systemd-bus-proxy:x:18074:0:99999:7:::
systemd-journal-gateway:x:18074:0:99999:7:::
systemd-journal-remote:x:18074:0:99999:7:::
systemd-journal-upload:x:18074:0:99999:7:::
systemd-network:x:18074:0:99999:7:::
systemd-resolve:x:18074:0:99999:7:::
systemd-timesync:x:18074:0:99999:7:::
nobody:x:18074:0:99999:7:::
syslog:!:18074::::::

```
Then we build the image via following command:    

```
# docker build -t goharbor/harbor-db:1.7.0-arm64 harbor-db/
```
We have to build all of the images:     

```
  docker build -t goharbor/chartmuseum-photon:v0.7.1-1.7.0-arm64 chartmuseum-photon/
  docker build -t goharbor/redis-photon:1.7.0-arm64 redis-photon/
  docker build -t goharbor/clair-photon:v2.0.7-1.7.0-arm64 clair-photon/
  docker build -t  goharbor/notary-server-photon:v0.6.1-1.7.0-arm64 notary-server-photon/
  docker build -t goharbor/notary-signer-photon:v0.6.1-1.7.0-arm64 notary-signer-photon
  docker build -t goharbor/harbor-registryctl:1.7.0-arm64 registry-photon
  docker build -t goharbor/registry-photon:v2.6.2-1.7.0-arm64 registry-photon/
  docker build -t goharbor/harbor-registryctl:1.7.0-arm64 harbor-registryctl/
  docker build -t goharbor/nginx-photon:1.7.0-arm64 nginx-photon/
  docker build -t goharbor/harbor-jobservice:1.7.0-arm64 harbor-jobservice/
  docker build -t goharbor/harbor-core:1.7.0-arm64 harbor-core/
  docker build -t goharbor/harbor-portal:1.7.0-arm64 harbor-portal/
  docker build -t goharbor/harbor-adminserver:1.7.0-arm64 harbor-adminserver/
  docker build -t goharbor/harbor-db:1.7.0-arm64 harbor-db/
```
