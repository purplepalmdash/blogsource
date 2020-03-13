+++
title= "UsingRedashForVisualization"
date = "2020-03-13T19:56:52+08:00"
description = "UsingRedashForVisualization"
keywords = ["Technology"]
categories = ["Technology"]
+++
### AIM
Using redash for representing Wuhan's nCov dataset.   

### Environment
Hardware/Software/OS is listed as following:     

```
kvm vm with 2cores and 3072MB memory plus 200G disk.
Using docker/docker-compose for redash website.
OS is ubuntu 18.04.3
IP: 192.168.137.100
```

### Redash bootstrap
Using the repository which located at:     
[https://github.com/getredash/setup](https://github.com/getredash/setup)    

You have to git clone this repository and run setup.sh under the directory. After setup the docker-compose file located in `/opt/redash`, if you want to quickly setup another docker/docker-compose based environment, simply copy the folder and the related docker images to remote machine's `/opt/redash`, up and running again.     

```
root@node:/opt/redash# ls
docker-compose.yml  env  postgres-data
root@node:/opt/redash# docker-compose up -d
Creating network "redash_default" with the default driver
Creating redash_redis_1    ... done
Creating redash_postgres_1 ... done
Creating redash_scheduled_worker_1 ... done
Creating redash_adhoc_worker_1     ... done
Creating redash_scheduler_1        ... done
Creating redash_server_1           ... done
Creating redash_nginx_1            ... done
```
The redash environment rested in 7 docker instance, could be examined using `docker ps`:     

```
root@node:/opt/redash# docker ps
CONTAINER ID        IMAGE                        COMMAND                  CREATED             STATUS              PORTS                         NAMES
1e701e0bb8d2        redash/nginx:latest          "nginx -g 'daemon of…"   6 minutes ago       Up 6 minutes        0.0.0.0:80->80/tcp, 443/tcp   redash_nginx_1
5c00c609baa6        redash/redash:8.0.0.b32245   "/app/bin/docker-ent…"   7 minutes ago       Up 6 minutes        0.0.0.0:5000->5000/tcp        redash_server_1
8dcd4c2db0f8        redash/redash:8.0.0.b32245   "/app/bin/docker-ent…"   7 minutes ago       Up 6 minutes        5000/tcp                      redash_scheduled_worker_1
5d1dbd77f4ce        redash/redash:8.0.0.b32245   "/app/bin/docker-ent…"   7 minutes ago       Up 6 minutes        5000/tcp                      redash_adhoc_worker_1
6d56d7cd9040        redash/redash:8.0.0.b32245   "/app/bin/docker-ent…"   7 minutes ago       Up 6 minutes        5000/tcp                      redash_scheduler_1
c8efb3ff7437        postgres:9.6-alpine          "docker-entrypoint.s…"   7 minutes ago       Up 7 minutes        5432/tcp                      redash_postgres_1
a78dd1b1727f        redis:5.0-alpine             "docker-entrypoint.s…"   7 minutes ago       Up 7 minutes        6379/tcp                      redash_redis_1
```
