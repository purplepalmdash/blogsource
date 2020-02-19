+++
title = "TipsOnPortusSync"
date = "2018-07-09T20:48:04+08:00"
description = "TipsOnPortusSync"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Portus Preparation
Install a new deployment node, with the `/var/lib/portus` empty, you could
manually delete this directory and create a new one.    

Then Configure the portus to setup the new namespace for holding the new
docker images, like following:    

![/images/2018_07_09_20_49_57_666x302.jpg](/images/2018_07_09_20_49_57_666x302.jpg)

Login in deployment node using following commands:    

```
[root@localhost ~]# cp /usr/local/compose/secrets/portus.crt /etc/pki/ca-trust/source/anchors/
[root@localhost ~]# update-ca-trust 
[root@localhost ~]# docker login portus.ddddddddd.com:5000
Username: kismatic
Password: 
Login Succeeded
```
### Kismatic Step
Get the latest release in:    

[https://github.com/apprenda/kismatic/releases](https://github.com/apprenda/kismatic/releases)    

prepare the kismatic env:    

```
# cd kismatic/
# tar xzvf kismatic-v1.12.0-linux-amd64.tar.gz 
# mv kismatic-v1.12.0-linux-amd64.tar.gz  ../
# ls
ansible  helm  kismatic  kubectl  provision
```
Planning the cluster configuration:    

```
# ./kismatic install plan
Plan your Kubernetes cluster:
=> Number of etcd nodes [3]: 1
=> Number of master nodes [2]: 1
=> Number of worker nodes [3]: 1
=> Number of ingress nodes (optional, set to 0 if not required) [2]: 0
=> Number of storage nodes (optional, set to 0 if not required) [0]: 0
=> Number of existing files or directories to be copied [0]: 0

Generating installation plan file template with: 
- 1 etcd nodes
- 1 master nodes
- 1 worker nodes
- 0 ingress nodes
- 0 storage nodes
- 0 files

Wrote plan file template to "kismatic-cluster.yaml"
Edit the plan file to further describe your cluster. Once ready, execute the "install validate" command to proceed.

```


