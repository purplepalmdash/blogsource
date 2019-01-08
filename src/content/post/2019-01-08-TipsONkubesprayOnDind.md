+++
title = "TipsOnKubesprayDind"
date = "2019-01-08T09:14:37+08:00"
description = "TipsOnKubesprayDind"
keywords = ["Linux"]
categories = ["Linux"]
+++
### Reference
Refers to :    

[https://github.com/kubernetes-sigs/kubespray/tree/master/contrib/dind](https://github.com/kubernetes-sigs/kubespray/tree/master/contrib/dind)

### Tips
Use custom docker images:    

```
# sudo docker pull ubuntu:16.04
# sudo docker run -it ubuntu:16.04 /bin/bash
root@d962689eb1ad:/etc/apt/apt.conf.d# echo>docker-clean
# docker commit  d962689eb1ad ubuntu:own
```
Thus the docker clean won't take effect, we could save all of the pkgs.    

Modify the images:    

```
$ pwd
/home/vagrant/kubespray-2.8.1/contrib/dind
$ vim ./group_vars/all/distro.yaml
    image: "ubuntu:own"
```
now follow the official guideline, be sure you modify the docker's definition.    

```
# vim roles/kubespray-defaults/defaults/main.yaml
  {%- if docker_version is version('17.05', '<') %}
  --graph={{ docker_daemon_graph }} {{ docker_log_opts }}
  {%- else %}
  --graph={{ docker_daemon_graph }} {{ docker_log_opts }}
  {%- endif %}
```
!!!But!!!, this will break the right logic of the kubespray itself.    


Command:    

```
cd contrib/dind
ansible-playbook -i hosts dind-cluster.yaml --extra-vars node_distro=ubuntu

cd ../..
CONFIG_FILE=inventory/local-dind/hosts.ini /tmp/kubespray.dind.inventory_builder.sh
ansible-playbook --become -e ansible_ssh_user=ubuntu -i inventory/local-dind/hosts.ini cluster.yml --extra-vars @contrib/dind/kubespray-dind.yaml --extra-vars bootstrap_os=ubuntu
```

### Meaning
The meaning of using dind for kubespray is: we could quickly get the offline
packages and docker images when kubespray release upgrades.    
