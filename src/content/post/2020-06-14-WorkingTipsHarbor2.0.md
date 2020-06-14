+++
title= "WorkingTipsHarbor2.0"
date = "2020-06-14T08:22:09+08:00"
description = "WorkingTipsHarbor2.0"
keywords = ["Technology"]
categories = ["Technology"]
+++
### Download Installation files
From following url:   

[https://github.com/goharbor/harbor/releases/download/v2.0.0/harbor-offline-installer-v2.0.0.tgz](https://github.com/goharbor/harbor/releases/download/v2.0.0/harbor-offline-installer-v2.0.0.tgz)    

### Config, Install
make new cert files via:   

```
TBD
```

Modify the `harbor.yml` file:    

```
5c5
< hostname: portus.fugouou.com
---
> hostname: reg.mydomain.com
15c15
<   port: 5000
---
>   port: 443
17,18c17,18
<   certificate: /data/cert/portus.crt
<   private_key: /data/cert/portus.key
---
>   certificate: /your/certificate/path
>   private_key: /your/private/key/path
29c29
< external_url: https://portus.fugouou.com:5000
---
> # external_url: https://reg.mydomain.com:8433
78c78
<   skip_update: true
---
>   skip_update: false
```
use the newly generated cert files:    

```
# mkdir -p /data/cert
# cp ***.crt ***.key /data/cert
```
Trivy offline database import:     

```
# ls /data/trivy-adapter/trivy/*
/data/trivy-adapter/trivy/metadata.json  /data/trivy-adapter/trivy/trivy.db

/data/trivy-adapter/trivy/db:
metadata.json  trivy.db
# chmod a+w -R /data/trivy-adapter/trivy/db
```

Install via:   

```
# ./install.sh --with-trivy --with-notary --with-chartmuseum
# docker ps
afe49ec2a626        goharbor/harbor-jobservice:v2.0.0      "/harbor/entrypoint.…"   14 minutes ago      Up 14 minutes (healthy)                                                                          harbor-jobservice
7ecf87cdc70b        goharbor/nginx-photon:v2.0.0           "nginx -g 'daemon of…"   14 minutes ago      Up 14 minutes (healthy)   0.0.0.0:4443->4443/tcp, 0.0.0.0:80->8080/tcp, 0.0.0.0:5000->8443/tcp   nginx
41fee7abd2a1        goharbor/notary-server-photon:v2.0.0   "/bin/sh -c 'migrate…"   14 minutes ago      Up 14 minutes                                                                                    notary-server
0f635fccd2fe        goharbor/notary-signer-photon:v2.0.0   "/bin/sh -c 'migrate…"   14 minutes ago      Up 14 minutes                                                                                    notary-signer
ebcd78417fdf        goharbor/harbor-core:v2.0.0            "/harbor/entrypoint.…"   14 minutes ago      Up 14 minutes (healthy)                                                                          harbor-core
28cca5aa3325        goharbor/trivy-adapter-photon:v2.0.0   "/home/scanner/entry…"   14 minutes ago      Up 14 minutes (healthy)   8080/tcp                                                               trivy-adapter
7823c38b71e9        goharbor/registry-photon:v2.0.0        "/home/harbor/entryp…"   14 minutes ago      Up 14 minutes (healthy)   5000/tcp                                                               registry
38b6bc813268        goharbor/harbor-portal:v2.0.0          "nginx -g 'daemon of…"   14 minutes ago      Up 14 minutes (healthy)   8080/tcp                                                               harbor-portal
ba6c5f9473b9        goharbor/redis-photon:v2.0.0           "redis-server /etc/r…"   14 minutes ago      Up 14 minutes (healthy)   6379/tcp                                                               redis
8ce7deffd0c8        goharbor/harbor-registryctl:v2.0.0     "/home/harbor/start.…"   14 minutes ago      Up 14 minutes (healthy)                                                                          registryctl
bd3085fb3b97        goharbor/chartmuseum-photon:v2.0.0     "./docker-entrypoint…"   14 minutes ago      Up 14 minutes (healthy)   9999/tcp                                                               chartmuseum
d3e90aa5d4d8        goharbor/harbor-db:v2.0.0              "/docker-entrypoint.…"   14 minutes ago      Up 14 minutes (healthy)   5432/tcp                                                               harbor-db
1b989340ff76        goharbor/harbor-log:v2.0.0             "/bin/sh -c /usr/loc…"   14 minutes ago      Up 14 minutes (healthy)   127.0.0.1:1514->10514/tcp                                              harbor-log
```
### Configure and use
In every node running docker doing(Ubuntu):    

```
# cp gwougou.crt  /usr/local/share/ca-certificates
# update-ca-certificates
# systemctl restart docker
# docker login -uadmin -pHarbor12345 xagowu.gowugoe.com
```

