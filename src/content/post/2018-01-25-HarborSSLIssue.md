+++
title = "HarborSSLIssue"
date = "2018-01-25T12:14:48+08:00"
description = "HarborSSLIssue"
keywords = ["Linux"]
categories = ["Linux"]
+++
### ArchLinux
Trust the ca certifications via:    

```
cp xxx.xxx.com.crt /etc/ca-certificates/trust-source/anchors/
update-ca-trust 
trust extract-compat
```

add `xxx.xxx.com` into `/etc/hosts`, then restart docker service, login with:    

```
# docker login xxx.xxx.com
```

### Ubuntu
Change the certifications via:    

```
# cp xxx.xxx.com.crt /usr/local/share/ca-certificates
# update-ca-certificates
# service docker restart
```
Now you could docker login into the new web server.    

### RHEL
update the ca, then docker login .   

```
cp certs/domain.crt /etc/pki/ca-trust/source/anchors/myregistrydomain.com.crt
update-ca-trust
```
