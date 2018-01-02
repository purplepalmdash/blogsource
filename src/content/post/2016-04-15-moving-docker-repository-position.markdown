---
categories: ["null"]
comments: true
date: 2016-04-15T15:55:15Z
title: Moving Docker Repository Position
url: /2016/04/15/moving-docker-repository-position/
---

Since the root partition is not so large, I have to change the default position of the
docker repository, following are the steps:    

### Make a Soft Link
Make a soft link to the docker repository via:    

```
$ sudo mkdir DockerRepo
$ sudo chown -R dash:dash DockerRepo
$ sudo chmod  777 -R DockerRepo
$ sudo tar -zcC /var/lib docker > /home/juju/DockerRepo/var_lib_docker-backup-$(date +%s).tar.gz
$ ls -l -h /home/juju/DockerRepo
$ sudo mv /var/lib/docker /home/juju/DockerRepo/
$ sudo ln -s /home/juju/DockerRepo/docker /var/lib/docker
$ sudo service docker restart
$ sudo service docker status
```
Notice: the `tar -zcC` will take a long time, be patient.    
In systemd like system, you should firstly stop docker.service, after moving,
restarting the docker.service.    
### Change Docker Parameters

```
Ubuntu/Debian: edit your /etc/default/docker file with the -g option: DOCKER_OPTS="-dns 8.8.8.8 -dns 8.8.4.4 -g /mnt"
```
