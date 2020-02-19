+++
title = "arm64KubesprayOfflineTips"
date = "2019-06-28T10:19:58+08:00"
description = "arm64KubesprayOfflineTips"
keywords = ["Linux"]
categories = ["Technology"]
+++
### Folder structure
Compare the downloaded source code to our offlined edition, make some changes.     

```
cluster.yml should added kube-deploy related items.   
ansible.cfg should be modified.
Added role/kube-deploy folder. 
scale.yml/upgrade-cluster.yml should be modified. 
Added deploy.key for easy deployment. 
roles/kubernetes-apps/ansible/defaults/main.yml, modified dashboard_skip_login condition
roles/kubernetes-apps/ansible/templates/dashboard.yml.j2: NodePort modification
roles/kubespray-defaults/defaults/main.yaml: enable_nodelocaldns:false(TBD)
roles/download/defaults/main.yml: download position, for example hyperkube/kubeadm/cni/calicoctl etc. 
/roles/kubernetes/master/templates/kubeadm-config.v1alpha3.yaml.j2: controllerManager listening port to 0.0.0.0
roles/kubernetes/master/tasks/kubeadm-upgrade.yml: upgrade items to --force
```

### (Todo) bootstrap.sh
Change the installation of ansible from apt-get to pip-cache

```
#!/bin/sh
## 
OS_ID=`cat /etc/os-release | grep VERSION_CODENAME | awk -F '=' {'print $2'}`
echo $OS_ID
# xenial use 1604, bionic use 1804
if [ "$OS_ID" = "xenial" ]; then
	sudo tar xJvf ./roles/kube-deploy/files/1604debs.tar.xz -C /usr/local/
else
	sudo tar xJvf ./roles/kube-deploy/files/1804debs.tar.xz -C /usr/local/
	sudo tar xJvf ./roles/kube-deploy/files/pip_ansible.tar.xz -C /usr/local/
fi
sudo echo "deb [trusted=yes] file:///usr/local/static ./">/etc/apt/sources.list
sudo apt-get update -y
# Install pip so we could use pip for installing ansible
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y python-pip
# Install ansible via ansible(version 2.8.1)
sudo pip install --no-index --find-links /usr/local/pip_ansible ansible
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y python-netaddr
```
### kube-deploy role
Added the offline role, and replace the files.   

Replace:    

```
nginx-autoindex.tar.xz 
kubeadm(arm version)
hyperkube(arm version)
cni-plugins-arm64-v0.6.0.tgz(arm version)
calicoctl(arm version)

```

### nginx-autoindex
Find the Dockerfile, and build the arm64 based docker images via following commands:     

```
# mkdir -p ~/code/autoindex
# vim Dockerfile
FROM nginx
MAINTAINER Jason Kingsbury

RUN sed -i 'N; s/root   \/usr\/share\/nginx\/html;\n        index  index.html index.htm;/root   \/usr\/share\/nginx\/html;\n        autoindex on;/' /etc/nginx/conf.d/default.conf
# sudo docker build -t jrelva/nginx-autoindex:latest .
# sudo docker  --name docker-nginx -p 7888:80 -d --restart=always -v `pwd`:/usr/share/nginx/html jrelva/nginx-autoindex
```
Save and xz the docker images:     

```
# sudo docker save jrelva/nginx-autoindex:latest>autoindex.tar; sudo xz autoindex.tar
```
Transfer the autoindex.tar.xz to folder.     

```
➜  files ls -l -h | grep autoindex
-rwxr-xr-x  1 dash dash  26M 5月   7 16:40 autoindex.tar.xz
➜  files pwd
/home/dash/Code/kubsprayarm64/roles/kube-deploy/files
➜  files ls -l -h | grep autoindex
-rwxr-xr-x  1 dash dash  23M 6月  28 10:59 autoindex.tar.xz
```
### secureregistryserver
Change the docker-compose file, also pull the arm based docker images:     

```
# docker pull registry:2
# docker pull nginx:latest
# docker save registry:2>regsitry.tar; xz registry.tar
# docker save nginx:latest>nginx19.tar; xz nginx19.tar

```

### Verification
Create a new virtual machine(18.04.2):    

```
$ qemu-img create -f qcow2 pure1804.qcow2 200G
```'
Install the system:    

![/images/2019_06_28_12_10_54_469x237.jpg](/images/2019_06_28_12_10_54_469x237.jpg)


### harbor offline
With the harbor-offline-installer-1.7.0-arm64.tgz we could quickly setup the offline harbor environment:    

```
# ls                                                                 
common                          docker-compose.clair.yml   docker-compose.yml         harbor.cfg  LICENSE              prepare                    
docker-compose.chartmuseum.yml  docker-compose.notary.yml  harbor.1.7.0-arm64.tar.gz  install.sh  open_source_license
# vim harbor.cfg
# ./install.sh --with-chartmuseum
# docker ps
.....
```

Login:    

![/images/2019_06_28_12_19_39_475x380.jpg](/images/2019_06_28_12_19_39_475x380.jpg)

Create user kubespray:    

![/images/2019_06_28_12_19_59_447x368.jpg](/images/2019_06_28_12_19_59_447x368.jpg)

Fill in user info:    

![/images/2019_06_28_12_20_28_555x469.jpg](/images/2019_06_28_12_20_28_555x469.jpg)

Create project:    

![/images/2019_06_28_12_21_45_549x348.jpg](/images/2019_06_28_12_21_45_549x348.jpg)

Projects:    

![/images/2019_06_28_12_22_02_667x296.jpg](/images/2019_06_28_12_22_02_667x296.jpg)

Add kubespray to kubesprayns as administrator:    

![/images/2019_06_28_12_22_42_718x295.jpg](/images/2019_06_28_12_22_42_718x295.jpg)

Now you could login with kubespray user:    

![/images/2019_06_28_12_23_52_936x419.jpg](/images/2019_06_28_12_23_52_936x419.jpg)

Now in docker-compose folder we just `docker-compose down` all of the service and backup our environment:    

```
# docker-compose down
Stopping nginx              ... done
Stopping harbor-jobservice  ... done
Stopping harbor-portal      ... done
Stopping harbor-core        ... done
Stopping redis              ... done
Stopping harbor-adminserver ... done
Stopping harbor-db          ... done
Stopping registry           ... done
Stopping registryctl        ... done
Stopping harbor-log         ... 
# docker save -o harbor.tar goharbor/chartmuseum-photon:v0.7.1-1.7.0-arm64 goharbor/redis-photon:1.7.0-arm64 goharbor/clair-photon:v2.0.7-1.7.0-arm64 goharbor/notary-server-photon:v0.6.1-1.7.0-arm64 goharbor/notary-signer-photon:v0.6.1-1.7.0-arm64 goharbor/harbor-registryctl:1.7.0-arm64 goharbor/registry-photon:v2.6.2-1.7.0-arm64 goharbor/nginx-photon:1.7.0-arm64 goharbor/harbor-log:1.7.0-arm64 goharbor/harbor-jobservice:1.7.0-arm64 goharbor/harbor-core:1.7.0-arm64 goharbor/harbor-portal:1.7.0-arm64 goharbor/harbor-adminserver:1.7.0-arm64 goharbor/harbor-db:1.7.0-arm64
# xz harbor.tar
```
