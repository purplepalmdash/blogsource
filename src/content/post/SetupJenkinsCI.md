+++
categories = ["Dev"]
date = "2016-07-13T19:28:30+08:00"
description = "Setup Jenkins CI Environment"
keywords = []
title = "SetupJenkinsCI"

+++
### AIM
Jenkins + Packer.io + GitLab + Gogs, for automatically building the virtual machine
images.    

### Jenkins Installation/Configuration
TBD

### GitLab
Refers to:    
[https://github.com/sameersbn/docker-gitlab](https://github.com/sameersbn/docker-gitlab)    

#### Image Preparation
Using docker for installing gitlab. First pull the docker image back via:    

```
$ docker pull sameersbn/gitlab:8.9.6
```
Also pull back the postgres and redis images, for we will link to these container's
services:    

```
$ docker pull sameersbn/postgresql:9.4-22
$ docker pull sameersbn/redis:latest
```
#### Run GitLab
Create the data directory for holding the data:     

```
$ mkdir -p /root/data/gitlab
$ cd /root/data/gitlab
$ mkdir redis postgresql gitlab
$ chmod 777 -R /root/data/gitlab/
```
Launch a postgresql container via:    

```
# docker run --name gitlab-postgresql -d \
     --env 'DB_NAME=gitlabhq_production' \
     --env 'DB_USER=gitlab' --env 'DB_PASS=password' \
     --env 'DB_EXTENSION=pg_trgm' \
     --volume /root/data/gitlab/postgresql:/var/lib/postgresql \
     sameersbn/postgresql:9.4-22
```
Launch a redis container via:    

```
# docker run --name gitlab-redis -d \
     --volume /root/data/gitlab/redis:/var/lib/redis \
     sameersbn/redis:latest
```

Launch the gitlab container:    

```
# docker run --name gitlab -d \
     --link gitlab-postgresql:postgresql --link gitlab-redis:redisio \
     --publish 10022:22 --publish 10080:80 \
     --env 'GITLAB_PORT=10080' --env 'GITLAB_SSH_PORT=10022' \
     --env 'GITLAB_SECRETS_DB_KEY_BASE=long-and-random-alpha-numeric-string' \
     --volume /root/data/gitlab/gitlab:/home/git/data \ 
     sameersbn/gitlab:8.9.6
```
Now visit `http://127.0.0.1:10080`, for setting up the password:    

![/images/2016_07_13_19_49_55_969x420.jpg](/images/2016_07_13_19_49_55_969x420.jpg)   

#### Systemd Services
For automatically startup the docker service in system boot, create the service items
in systemd, listed in following:    

Create the gitlab service:    

```
$ vim /usr/lib/systemd/system/gitlab.service
[Unit]
Description=gitlab
Requires=docker.service
After=docker.service

[Service]
Restart=always
ExecStart=/usr/bin/docker start -a gitlab
ExecStop=/usr/bin/docker stop -t 2 gitlab

[Install]
WantedBy=multi-user.target
``` 
Also create another 2 service named `gitlab-redis.service` and
`gitlab-postgres.service`, then enable the service via:     

```
# systemctl enable gitlabpostgres.service
# systemctl enable gitlabredis.service
# systemctl enable gitlab.service
```
Then next time when system reboot, these service will automatically restart.    
